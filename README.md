# Echo Marketing Studio — Backend Ready Struktura

## 📁 Struktura projektu

```
echomedia/
├── index.html          ← Frontend (gotowy)
├── style.css           ← Stylowanie (gotowy)
├── img/
│   ├── logo.png
│   ├── photo example.png
│   ├── pin.svg
│   ├── card.svg
│   └── backgroung.svg
│
├── api/                ← [BACKEND — do stworzenia]
│   ├── contact.php     ← lub contact.js (Node.js)
│   └── .env.example
│
└── README.md           ← ten plik
```

---

## 🔌 Integracja backendu — formularz kontaktowy

W pliku `index.html`, funkcja JavaScript (linia ~`await new Promise(...)`)
jest przygotowana na zamianę na prawdziwe żądanie HTTP:

### Opcja A — PHP (hosting współdzielony)

```php
// api/contact.php
<?php
header('Content-Type: application/json');
header('Access-Control-Allow-Origin: *');

if ($_SERVER['REQUEST_METHOD'] !== 'POST') {
    http_response_code(405);
    exit(json_encode(['error' => 'Method not allowed']));
}

$data   = json_decode(file_get_contents('php://input'), true);
$name   = htmlspecialchars($data['name'] ?? '');
$email  = filter_var($data['email'] ?? '', FILTER_SANITIZE_EMAIL);
$company = htmlspecialchars($data['company'] ?? '');
$message = htmlspecialchars($data['message'] ?? '');

// Walidacja
if (!$name || !filter_var($email, FILTER_VALIDATE_EMAIL) || !$message) {
    http_response_code(400);
    exit(json_encode(['error' => 'Brakuje wymaganych pól']));
}

// Wysyłka maila
$to      = 'kontakt@echomedia.pl';   // ← zmień na swój email
$subject = "Nowe zapytanie od: $name";
$body    = "Imię: $name\nEmail: $email\nFirma: $company\n\nWiadomość:\n$message";
$headers = "From: noreply@echomedia.pl\r\nReply-To: $email";

$sent = mail($to, $subject, $body, $headers);

echo json_encode(['success' => $sent]);
```

### Opcja B — Node.js / Express

```js
// api/contact.js
const express  = require('express');
const nodemailer = require('nodemailer');
const router   = express.Router();

router.post('/contact', async (req, res) => {
  const { name, email, company, message } = req.body;
  if (!name || !email || !message)
    return res.status(400).json({ error: 'Brakuje wymaganych pól' });

  const transporter = nodemailer.createTransport({
    host: process.env.SMTP_HOST,
    port: 465,
    secure: true,
    auth: { user: process.env.SMTP_USER, pass: process.env.SMTP_PASS },
  });

  await transporter.sendMail({
    from: `"Echo Studio" <${process.env.SMTP_USER}>`,
    to:   'kontakt@echomedia.pl',
    replyTo: email,
    subject: `Nowe zapytanie od: ${name}`,
    text: `Imię: ${name}\nEmail: ${email}\nFirma: ${company}\n\n${message}`,
  });

  res.json({ success: true });
});

module.exports = router;
```

### Podmiana w `index.html`

Zamień komentarz `← replace with fetch(...)` na:

```js
const response = await fetch('/api/contact', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    name, email,
    company: document.getElementById('company').value.trim(),
    message
  })
});
if (!response.ok) throw new Error('Błąd serwera');
```

---

## 🌐 Deploy / Upload na serwer

### Opcja 1 — Zwykły hosting (FTP)
1. Wgraj cały folder `echomedia/` przez FTP (FileZilla, WinSCP)
2. Upewnij się że `index.html` jest w katalogu głównym (`public_html/`)
3. Skonfiguruj domenę → folder

### Opcja 2 — cPanel + File Manager
1. Wejdź w cPanel → File Manager
2. Wgraj pliki do `public_html/`

### Opcja 3 — Vercel / Netlify (frontend only)
```bash
# Netlify CLI
npm install -g netlify-cli
netlify deploy --dir=. --prod

# Vercel
npm install -g vercel
vercel --prod
```

### Opcja 4 — VPS z Nginx
```nginx
server {
    listen 80;
    server_name echomedia.pl www.echomedia.pl;
    root /var/www/echomedia;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /api/ {
        proxy_pass http://localhost:3000;
    }
}
```

---

## ✅ Checklist przed deployem

- [ ] Zmień adres email w backendzie na swój
- [ ] Skonfiguruj SMTP (lub użyj php `mail()`)
- [ ] Ustaw zmienne środowiskowe w `.env`
- [ ] Dodaj HTTPS (Let's Encrypt / certbot)
- [ ] Sprawdź formularz na mobile
- [ ] Dodaj Google Analytics lub Matomo (opcjonalnie)
- [ ] Ustaw meta OG image (ogimage.jpg)
