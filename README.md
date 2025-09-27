# Nest Auth Prisma Boilerplate 🔐

A boilerplate for authentication built with **NestJS**, **Prisma**, and **PostgreSQL**, designed to be a solid starting point for full-stack apps.  
It supports email/password authentication, Google OAuth, refresh tokens, and 2FA hooks, with cookies-based session handling.  

---

## ✨ Features

- **User Registration** with email verification link  
- **Login** with access + refresh tokens (JWT)  
- **Google OAuth** ready (React & Flutter snippets provided)  
- **2FA (Google Authenticator)** verification flow included  
- **Password Reset** via email link  
- **Cookies for Auth** (HTTP-only access & refresh cookies)  
- **Prisma ORM** with PostgreSQL schema  
- **Extendable** → Add phone/email OTP auth, or custom integrations  

---

## 🛠️ Tech Stack

- **NestJS** – Backend framework  
- **Prisma** – ORM  
- **PostgreSQL** – Database  
- **JWT** – Authentication tokens  
- **Nodemailer** – Email handling  
- **Google OAuth** – Third-party login  

---

## 📂 Project Setup

### Prerequisites
- Node.js v18+  
- pnpm (`npm install -g pnpm`)  
- PostgreSQL v13+ running locally or hosted  
- Google OAuth credentials  
- App password for Gmail (see [Gmail App Passwords](https://support.google.com/accounts/answer/185833))  

### Installation
```bash
git clone <repo-url>
cd nest-auth-prisma-boilerplate
pnpm install
````

### Prisma Setup

```bash
pnpm prisma generate
pnpm prisma db push
```

### Environment Variables

Copy `.env.example` → `.env` and configure:

```env
DATABASE_URL="postgresql://postgres:password@localhost:5432/db_name"
JWT_SECRET="your-jwt-secret"
NODE_ENV="development"

GOOGLE_CLIENT_ID="your-google-client-id"
GOOGLE_CLIENT_SECRET="your-google-client-secret"

EMAIL_USER="your-email"
EMAIL_PASS="your-app-password"
```

---

## 🔑 API Routes

| Route                  | Method | Description                                                   |
| ---------------------- | ------ | ------------------------------------------------------------- |
| `/auth/register`       | POST   | Register new user, hash password, send verification email     |
| `/auth/login`          | POST   | Login with email/password → set access & refresh tokens (JWT) |
| `/auth/google`         | POST   | Google OAuth login (send `code`)                              |
| `/auth/refresh`        | GET    | Refresh expired access token using refresh token              |
| `/auth/verify_2fa`     | POST   | Verify Google Authenticator 2FA code                          |
| `/auth/reset_password` | POST   | Reset password via email link + new password                  |

---

## 📐 Prisma Schema (Snippet)

```prisma
model User {
  id String @id @default(cuid())
  name String
  email String @unique
  password String
  status UserStatus @default(PENDING)
  two_factor_secret String?
  is2FAEnabled Boolean @default(false)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  @@index([email])
}

model Token {
  id String @id @default(cuid())
  user_id String
  user User @relation(fields: [user_id], references: [id], onDelete: Cascade)
  token String
  type TokenType
  createdAt DateTime @default(now())
  expiresAt DateTime
  @@index([user_id])
  @@index([token])
}

enum UserStatus {
  PENDING
  ACTIVE
}

enum TokenType {
  CONFIRMATION
  REFRESH
}
```

---

## 🧩 Frontend Integration (Google OAuth)

### React (Next.js)

```tsx
import { useEffect } from "react";

export default function GoogleLogin() {
  const handleGoogleLogin = () => {
    const clientId = process.env.NEXT_PUBLIC_GOOGLE_CLIENT_ID!;
    const redirectUri = "http://localhost:3000/auth/callback";

    const url = `https://accounts.google.com/o/oauth2/v2/auth?client_id=${clientId}&redirect_uri=${redirectUri}&response_type=code&scope=openid%20email%20profile`;

    window.location.href = url;
  };

  return (
    <button onClick={handleGoogleLogin}>
      Continue with Google
    </button>
  );
}

// Callback page
import { useRouter } from "next/router";
import { useEffect } from "react";

export default function GoogleCallback() {
  const router = useRouter();

  useEffect(() => {
    const { code } = router.query;
    if (code) {
      fetch("http://localhost:5000/auth/google", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ code }),
        credentials: "include",
      }).then(() => router.push("/dashboard"));
    }
  }, [router.query]);

  return <p>Logging in...</p>;
}
```

---

### Flutter (Dart)

```dart
import 'package:flutter/material.dart';
import 'package:url_launcher/url_launcher.dart';
import 'package:http/http.dart' as http;

class GoogleLoginButton extends StatelessWidget {
  final String clientId = "YOUR_GOOGLE_CLIENT_ID";
  final String redirectUri = "http://localhost:3000/auth/callback";

  void _loginWithGoogle() async {
    final authUrl =
        "https://accounts.google.com/o/oauth2/v2/auth?client_id=$clientId&redirect_uri=$redirectUri&response_type=code&scope=openid%20email%20profile";
    if (await canLaunch(authUrl)) {
      await launch(authUrl);
    }
  }

  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: _loginWithGoogle,
      child: Text("Continue with Google"),
    );
  }
}

// On callback screen, grab "code" from redirected URL and POST to backend
Future<void> handleCallback(String code) async {
  final response = await http.post(
    Uri.parse("http://localhost:5000/auth/google"),
    headers: {"Content-Type": "application/json"},
    body: '{"code": "$code"}',
  );
  if (response.statusCode == 200) {
    print("Logged in successfully");
  }
}
```

---

## 🚀 Future Improvements

* ✅ Add phone number OTP auth (SMS/WhatsApp)
* ✅ Add email OTP login (alternative to Google)
* ✅ Write unit & integration tests
* ✅ Improve password reset with signed tokens in URL

---

## 📄 License

MIT License. Use, modify, and contribute freely.

```
```
