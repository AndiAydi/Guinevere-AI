# Guinevere AI - Deployment Guide

Panduan lengkap untuk menjalankan Guinevere AI di hosting sendiri.

## Prasyarat

- **Node.js**: v22.13.0 atau lebih tinggi
- **pnpm**: v10.4.1 atau lebih tinggi
- **MySQL/TiDB**: Database server yang kompatibel
- **OpenAI API Key**: Untuk integrasi LLM

## Instalasi Lokal

### 1. Setup Environment Variables

Buat file `.env.local` di root project:

```bash
# Database
DATABASE_URL=mysql://username:password@localhost:3306/guinevere_ai

# OAuth (Manus)
VITE_APP_ID=your_app_id
OAUTH_SERVER_URL=https://api.manus.im
VITE_OAUTH_PORTAL_URL=https://portal.manus.im
JWT_SECRET=your_jwt_secret_key_here

# Owner Info
OWNER_OPEN_ID=your_owner_id
OWNER_NAME=Your Name

# Manus Built-in APIs
BUILT_IN_FORGE_API_URL=https://api.manus.im
BUILT_IN_FORGE_API_KEY=your_forge_api_key
VITE_FRONTEND_FORGE_API_KEY=your_frontend_forge_api_key
VITE_FRONTEND_FORGE_API_URL=https://api.manus.im

# Analytics (Optional)
VITE_ANALYTICS_ENDPOINT=your_analytics_endpoint
VITE_ANALYTICS_WEBSITE_ID=your_website_id

# App Config
VITE_APP_TITLE=Guinevere AI
VITE_APP_LOGO=https://your-logo-url.png
```

### 2. Install Dependencies

```bash
pnpm install
```

### 3. Setup Database

```bash
# Generate migration files
pnpm drizzle-kit generate

# Apply migrations to database
pnpm drizzle-kit migrate
```

### 4. Build Project

```bash
pnpm build
```

### 5. Run Development Server

```bash
pnpm dev
```

Server akan berjalan di `http://localhost:3000`

### 6. Run Production Server

```bash
pnpm start
```

## Deployment ke Hosting

### Option 1: Railway

```bash
# Install Railway CLI
npm i -g @railway/cli

# Login
railway login

# Init project
railway init

# Deploy
railway up
```

### Option 2: Render

1. Push code ke GitHub
2. Connect repository ke Render
3. Set environment variables di Render dashboard
4. Deploy

### Option 3: Vercel (Frontend Only)

```bash
# Install Vercel CLI
npm i -g vercel

# Deploy
vercel
```

### Option 4: Docker

Buat `Dockerfile`:

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package.json pnpm-lock.yaml ./
RUN npm install -g pnpm && pnpm install

COPY . .

RUN pnpm build

EXPOSE 3000

CMD ["pnpm", "start"]
```

Build dan run:

```bash
docker build -t guinevere-ai .
docker run -p 3000:3000 -e DATABASE_URL=your_db_url guinevere-ai
```

## Database Setup

### MySQL/TiDB Connection

```sql
CREATE DATABASE guinevere_ai;
USE guinevere_ai;
```

Kemudian jalankan migrations:

```bash
pnpm drizzle-kit migrate
```

### Verify Database

```bash
mysql -u username -p -h localhost guinevere_ai -e "SHOW TABLES;"
```

Seharusnya ada 4 tables:
- `users`
- `conversations`
- `userRelationships`
- `userPreferences`

## Project Structure

```
guinevere-ai/
├── client/                 # Frontend React app
│   ├── src/
│   │   ├── components/    # React components
│   │   ├── pages/         # Page components
│   │   ├── lib/           # Utilities & tRPC client
│   │   ├── contexts/      # React contexts
│   │   └── App.tsx        # Main app component
│   ├── public/            # Static files
│   └── index.html         # HTML template
├── server/                # Backend Express server
│   ├── personality/       # Guinevere personality system
│   ├── problemSolving/    # Problem-solving engine
│   ├── services/          # Business logic services
│   ├── middleware/        # Express middleware
│   ├── routers/           # tRPC routers
│   └── _core/             # Core infrastructure
├── drizzle/               # Database schema & migrations
├── shared/                # Shared types & constants
├── storage/               # S3 storage helpers
├── package.json           # Dependencies
├── tsconfig.json          # TypeScript config
├── vite.config.ts         # Vite config
└── drizzle.config.ts      # Drizzle config
```

## Key Files

| File | Purpose |
|------|---------|
| `server/personality/constants.ts` | Guinevere personality definition |
| `server/personality/systemPrompt.ts` | System prompt generator |
| `server/personality/manager.ts` | Personality state management |
| `server/problemSolving/` | Problem-solving engine |
| `server/services/chatService.ts` | Chat processing with OpenAI |
| `drizzle/schema.ts` | Database schema |
| `package.json` | Dependencies & scripts |

## Environment Variables Explained

| Variable | Purpose |
|----------|---------|
| `DATABASE_URL` | MySQL connection string |
| `VITE_APP_ID` | OAuth application ID |
| `JWT_SECRET` | Session cookie signing key |
| `BUILT_IN_FORGE_API_KEY` | Manus API authentication |
| `VITE_APP_TITLE` | Application title |

## Troubleshooting

### Database Connection Error

```
Error: connect ECONNREFUSED 127.0.0.1:3306
```

**Solution**: Pastikan MySQL server running dan DATABASE_URL benar.

### Missing Dependencies

```
Error: Cannot find module 'xxx'
```

**Solution**: Jalankan `pnpm install` ulang.

### TypeScript Errors

```
error TS2307: Cannot find module
```

**Solution**: Jalankan `pnpm check` untuk verify TypeScript configuration.

### Port Already in Use

```
Error: listen EADDRINUSE :::3000
```

**Solution**: Ubah port di `server/_core/index.ts` atau kill process yang menggunakan port 3000.

## Testing

Jalankan semua tests:

```bash
pnpm test
```

Run specific test:

```bash
pnpm test server/personality/manager.test.ts
```

## Performance Tips

1. **Enable Response Caching**: Gunakan Redis untuk cache responses
2. **Database Indexing**: Add indexes pada frequently queried columns
3. **CDN for Assets**: Host static files di CDN
4. **Rate Limiting**: Sudah built-in, adjust di `server/middleware/rateLimiter.ts`
5. **Monitoring**: Setup error tracking dengan Sentry atau similar

## Security Checklist

- [ ] Change `JWT_SECRET` ke value yang kuat
- [ ] Setup HTTPS/SSL certificate
- [ ] Enable CORS dengan whitelist domains
- [ ] Setup rate limiting (sudah ada)
- [ ] Sanitize user inputs
- [ ] Setup database backups
- [ ] Monitor API usage

## Maintenance

### Database Backups

```bash
# Backup
mysqldump -u username -p guinevere_ai > backup.sql

# Restore
mysql -u username -p guinevere_ai < backup.sql
```

### Update Dependencies

```bash
pnpm update
pnpm audit
```

### Monitor Logs

```bash
# Development
tail -f .manus-logs/devserver.log

# Production
tail -f logs/app.log
```

## Support & Resources

- **Documentation**: Lihat README.md
- **API Docs**: Lihat `server/routers.ts` untuk tRPC procedures
- **Database Schema**: Lihat `drizzle/schema.ts`
- **Tests**: Lihat `server/**/*.test.ts` untuk contoh

## License

Guinevere AI © 2026. All rights reserved.

---

**Last Updated**: April 2026
**Version**: 2.1
