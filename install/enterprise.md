# Enterprise Deployment

## Self-Hosted Setup

### Requirements
- Node.js 16+ or Docker
- Vercel / AWS / DigitalOcean account
- Git

### Steps

1. **Fork the repo:** https://github.com/adifydigitalnoida-del/tokenoptim
2. **Clone your fork:** `git clone https://github.com/YOUR-USERNAME/tokenoptim`
3. **Deploy to Vercel:**
   - Go to vercel.com
   - Import your GitHub fork
   - Click Deploy
   - Done in 2 minutes

OR **Deploy with Docker:**
```bash
docker build -t tokenoptim .
docker run -p 3000:3000 tokenoptim
```

OR **Deploy with Node.js:**
```bash
npm install
npm start
```

### Features
- Full data control
- Zero telemetry
- Custom branding
- Team management
- SSO integration

[Back to TokenOptim](/)
