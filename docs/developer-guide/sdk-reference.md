# SDK Reference

There's no separately published SDK package. `mergepay-web` talks to the
API through one typed client in [`src/lib/api.ts`](https://github.com/mergepay/mergepay-web/blob/main/src/lib/api.ts),
and every component calls through it. This page documents that real
client, not an invented wrapper.

## The `request` helper

All calls go through one generic function. It attaches the stored JWT as a
`Bearer` token, serializes a `json` body, clears the session on a `401`,
and throws a typed error on any non-2xx response.

```typescript
async function request<T>(
  path: string,
  options: RequestInit & { json?: unknown } = {}
): Promise<T> {
  const headers: Record<string, string> = { ...(options.headers as Record<string, string>) };
  const token = getToken();
  if (token) headers.Authorization = `Bearer ${token}`;

  let body = options.body;
  if (options.json !== undefined) {
    headers["Content-Type"] = "application/json";
    body = JSON.stringify(options.json);
  }

  const res = await fetch(`${API_URL}${path}`, { ...options, headers, body });

  if (res.status === 401 && token) {
    useAuth.getState().clear(); // session expired -> auth guard redirects to /login
  }

  if (!res.ok) {
    let code = "unknown";
    let message = `Request failed (${res.status})`;
    try {
      const data = await res.json();
      code = data?.error?.code ?? code;
      message = data?.error?.message ?? message;
    } catch {
      // non-JSON error body
    }
    throw new ApiRequestError(res.status, code, message);
  }

  if (res.status === 204) return undefined as T;
  return (await res.json()) as T;
}
```

## Error type

Every failed request throws an `ApiRequestError` carrying the HTTP status
and the API's error `code` (the same codes listed in the
[API Reference](api-reference.md)), so callers can branch on the code
rather than parsing message strings.

```typescript
export class ApiRequestError extends Error {
  code: string;
  status: number;
  constructor(status: number, code: string, message: string) {
    super(message);
    this.name = "ApiRequestError";
    this.status = status;
    this.code = code;
  }
}
```

Handling a specific case — for example, a missing trustline during
settlement:

```typescript
try {
  await api.confirmSettlement(settlementId, { signedXdr });
} catch (e) {
  if (e instanceof ApiRequestError && e.code === "missing_trustline") {
    // prompt the user to add the USDC trustline in Freighter, then retry
  } else {
    throw e;
  }
}
```

## The `api` object

Methods are grouped by area. Each returns a typed promise; the type
arguments are the response shapes declared in
[`src/lib/types.ts`](https://github.com/mergepay/mergepay-web/blob/main/src/lib/types.ts).

### Auth
```typescript
api.authChallenge(account: string)      // POST /auth/challenge -> { transaction }
api.authVerify(transaction: string)     // POST /auth/verify    -> { token, user }
api.authLogout()                         // POST /auth/logout
api.me()                                 // GET  /me
api.updateMe(data)                       // PATCH /me
```

### Groups
```typescript
api.createGroup(data)                    // POST /groups
api.listGroups()                         // GET  /groups
api.getGroup(id)                         // GET  /groups/:id
api.createInvite(groupId, data?)         // POST /groups/:id/invite
api.joinGroup(code)                      // POST /groups/join
api.leaveGroup(groupId)                  // POST /groups/:id/leave
api.archiveGroup(groupId)                // POST /groups/:id/archive
```

### Expenses
```typescript
api.createExpense(groupId, data)         // POST /groups/:id/expenses
api.listExpenses(groupId)                // GET  /groups/:id/expenses
api.getExpense(id)                       // GET  /expenses/:id
api.updateExpense(id, data)              // PATCH /expenses/:id  (metadata only)
api.deleteExpense(id)                    // DELETE /expenses/:id
```

### Settlement
```typescript
api.settleExpense(expenseId, data?)      // POST /expenses/:id/settle       -> { xdr, ... }
api.createSettlement(groupId, data)      // POST /groups/:id/settlements    -> { xdr, ... }
api.confirmSettlement(settlementId, { signedXdr })  // POST /settlements/:id/confirm
api.getBalances(groupId)                 // GET  /groups/:id/balances
api.getLedger(groupId)                   // GET  /groups/:id/ledger
```

### Treasury
```typescript
api.enableTreasury(groupId, data)        // POST /groups/:id/treasury/enable
api.treasuryInfo(groupId)                // GET  /groups/:id/treasury
api.treasuryDeposit(groupId, data)       // POST /groups/:id/treasury/deposit
api.treasuryWithdraw(groupId, data)      // POST /groups/:id/treasury/withdraw
api.confirmTreasuryTx(txId, { signedXdr })  // POST /treasury-transactions/:id/confirm
api.treasuryHistory(groupId)             // GET  /groups/:id/treasury/history
```

### Anchors, history, uploads
```typescript
api.listAnchors()                        // GET  /anchors
api.anchorDeposit(data)                  // POST /anchors/deposit  -> interactive URL
api.anchorWithdraw(data)                 // POST /anchors/withdraw -> interactive URL
api.anchorComplete(sessionId, data)      // POST /anchors/sessions/:id/complete
api.anchorSessions()                     // GET  /anchors/sessions
api.history()                            // GET  /history
api.uploadReceipt(file)                  // POST /uploads (multipart)
```

A settlement is always two calls: build (`settleExpense` or
`createSettlement`) returns an unsigned XDR, then `confirmSettlement`
submits the wallet-signed XDR. See [Settlement Mechanics](../protocol/settlement-mechanics.md)
for what happens between them.
