# Unofficial Brasil: Guia Técnico de Segurança e SEO (Bootstrap)

**Modelo:** Faça você mesmo (sem contratações externas)  
**Nível:** Intermediário a Avançado  
**Tempo estimado:** 40 horas (distribuído em 3 semanas)

---

## SUMÁRIO EXECUTIVO

Este guia fornece implementação prática de:
1. **Segurança OWASP Top 10** (auditoria interna, sem pentest pago)
2. **SEO técnico** (sem consultoria, apenas ferramentas gratuitas)
3. **Performance optimization** (Core Web Vitals)
4. **Monitoring & alerting** (ferramentas free tier)

**Tudo executável por você, sem custos externos.**

---

## PARTE 1: SEGURANÇA (OWASP TOP 10)

### Checklist Executável

Execute estes testes você mesmo. Para cada teste que falhar, há uma solução de código.

#### 1. Broken Access Control

**Teste:**
```bash
# 1. Criar 2 contas de teste
# User A: phone = 5511111111111
# User B: phone = 5511222222222

# 2. Login como User A, pegar token
# DevTools → Application → localStorage → auth_token

# 3. Tentar acessar dados do User B
curl https://unofficialbrasil.com.br/api/identity/user-b-id \
  -H "Authorization: Bearer [user-a-token]"

# ESPERADO: 403 Forbidden
# SE RETORNAR dados do User B → VULNERABILIDADE CRÍTICA
```

**Solução (se falhou):**
```typescript
// app/api/identity/[id]/route.ts
import { getSession } from '@/lib/auth';

export async function GET(
  req: Request,
  { params }: { params: { id: string } }
) {
  // CRÍTICO: Sempre verificar ownership
  const session = await getSession(req);
  const requestedUserId = parseInt(params.id);
  
  if (session.userId !== requestedUserId && !session.isAdmin) {
    return Response.json(
      { error: 'Access denied' },
      { status: 403 }
    );
  }
  
  // Seguro prosseguir
  const user = await db.identity.findUnique({
    where: { id: requestedUserId }
  });
  
  return Response.json(user);
}
```

**Aplicar em TODOS endpoints que retornam dados de usuário:**
- `/api/identity/:id`
- `/api/memberships`
- `/api/communities/:id/members`
- `/api/gamification/progress`

#### 2. SQL Injection

**Teste:**
```bash
# Tentar injeção no campo de busca
curl -X POST https://unofficialbrasil.com.br/api/communities/search \
  -H "Content-Type: application/json" \
  -d '{"query": "test OR 1=1"}'

# ESPERADO: Resultados filtrados corretamente (não todas comunidades)
# SE RETORNAR todas comunidades → VULNERABILIDADE CRÍTICA

# Teste 2: Telefone com SQL
curl -X POST https://unofficialbrasil.com.br/api/auth/signup \
  -H "Content-Type: application/json" \
  -d '{"phone": "5511999999999; DROP TABLE identities;--"}'

# ESPERADO: 400 Bad Request (validação rejeita)
# SE EXECUTAR comando SQL → VULNERABILIDADE CRÍTICA
```

**Solução: SEMPRE usar Prisma**

```typescript
// ❌ NUNCA FAÇA ISSO
async function searchCommunities(query: string) {
  const sql = `SELECT * FROM communities WHERE name LIKE '%${query}%'`;
  return await db.$queryRawUnsafe(sql);
}

// ✅ SEMPRE FAÇA ASSIM
async function searchCommunities(query: string) {
  // Prisma automaticamente sanitiza
  return await db.community.findMany({
    where: {
      name: {
        contains: query, // Safe: Prisma usa parameterized queries
      }
    }
  });
}
```

**Regra de ouro:** Se você vê `$queryRawUnsafe` ou string concatenation em SQL, **DELETE e reescreva com Prisma**.

#### 3. Rate Limiting

**Teste:**
```bash
# Testar limite de OTP requests
for i in {1..10}; do
  echo "Request $i"
  curl -X POST https://unofficialbrasil.com.br/api/auth/otp-send \
    -H "Content-Type: application/json" \
    -d '{"phone":"5511999999999"}'
  sleep 1
done

# ESPERADO: Após 3-5 requests, retorna 429 Too Many Requests
# SE ACEITAR todos 10 → VULNERABILIDADE (DDoS attack vector)
```

**Implementação Rate Limiting (Redis):**

```typescript
// lib/rateLimit.ts
import Redis from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

export async function rateLimit(
  identifier: string, // IP ou user ID
  action: string,     // ex: "otp-send"
  limit: number,      // ex: 3
  window: number      // segundos, ex: 600 (10 min)
): Promise<{ allowed: boolean; remaining: number }> {
  const key = `rate:${action}:${identifier}`;
  
  const count = await redis.incr(key);
  
  if (count === 1) {
    await redis.expire(key, window);
  }
  
  const allowed = count <= limit;
  const remaining = Math.max(0, limit - count);
  
  return { allowed, remaining };
}

// Uso em API route
export async function POST(req: Request) {
  const ip = req.headers.get('x-forwarded-for') || 
             req.headers.get('x-real-ip') || 
             'unknown';
  
  const { allowed, remaining } = await rateLimit(
    ip,
    'otp-send',
    3,    // 3 tentativas
    600   // em 10 minutos
  );
  
  if (!allowed) {
    return Response.json(
      { 
        error: 'Muitas tentativas. Tente novamente em 10 minutos.',
        retryAfter: 600 
      },
      { 
        status: 429,
        headers: {
          'Retry-After': '600'
        }
      }
    );
  }
  
  // Prosseguir com OTP send...
  return Response.json({ 
    success: true,
    remaining 
  });
}
```

**Configurar rate limits para:**

```typescript
// config/rateLimits.ts
export const RATE_LIMITS = {
  'otp-send': { limit: 3, window: 600 },        // 3 por 10 min
  'otp-verify': { limit: 5, window: 600 },      // 5 por 10 min
  'signup': { limit: 5, window: 3600 },         // 5 por 1 hora
  'login': { limit: 10, window: 600 },          // 10 por 10 min
  'contact-form': { limit: 2, window: 3600 },   // 2 por 1 hora
  'api-general': { limit: 100, window: 60 },    // 100 por 1 min
};
```

#### 4. SSL/TLS Configuration

**Teste:**
```bash
# 1. Verificar SSL grade
# Abra: https://www.ssllabs.com/ssltest/analyze.html?d=unofficialbrasil.com.br
# ESPERADO: Grade A ou A+

# 2. Verificar redirect HTTP → HTTPS
curl -I http://unofficialbrasil.com.br
# ESPERADO: 301 Moved Permanently
#           Location: https://unofficialbrasil.com.br

# 3. Verificar HSTS header
curl -I https://unofficialbrasil.com.br | grep -i strict-transport-security
# ESPERADO: Strict-Transport-Security: max-age=31536000
```

**Configuração Nginx (Let's Encrypt):**

```nginx
# /etc/nginx/sites-available/unofficialbrasil.com.br

# HTTPS Server
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name unofficialbrasil.com.br www.unofficialbrasil.com.br;

    # SSL Certificate (Let's Encrypt)
    ssl_certificate /etc/letsencrypt/live/unofficialbrasil.com.br/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/unofficialbrasil.com.br/privkey.pem;
    
    # SSL Configuration (Mozilla Intermediate - June 2024)
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384';
    ssl_prefer_server_ciphers off;
    
    # OCSP Stapling
    ssl_stapling on;
    ssl_stapling_verify on;
    ssl_trusted_certificate /etc/letsencrypt/live/unofficialbrasil.com.br/chain.pem;
    
    # Security Headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
    add_header X-Frame-Options "DENY" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Permissions-Policy "geolocation=(), microphone=(), camera=()" always;
    
    # Content Security Policy
    add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' https://www.googletagmanager.com https://www.google-analytics.com; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self' data:; connect-src 'self' https://api.unofficialbrasil.com.br https://www.google-analytics.com; frame-ancestors 'none';" always;
    
    # Proxy to Next.js
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
    
    # Gzip Compression
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types text/plain text/css text/xml text/javascript application/json application/javascript application/xml+rss application/rss+xml font/truetype font/opentype application/vnd.ms-fontobject image/svg+xml;
}

# HTTP → HTTPS Redirect
server {
    listen 80;
    listen [::]:80;
    server_name unofficialbrasil.com.br www.unofficialbrasil.com.br;
    
    return 301 https://unofficialbrasil.com.br$request_uri;
}

# www → non-www Redirect
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name www.unofficialbrasil.com.br;
    
    ssl_certificate /etc/letsencrypt/live/unofficialbrasil.com.br/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/unofficialbrasil.com.br/privkey.pem;
    
    return 301 https://unofficialbrasil.com.br$request_uri;
}
```

**Instalar Let's Encrypt:**

```bash
# 1. Instalar Certbot
sudo apt update
sudo apt install certbot python3-certbot-nginx

# 2. Obter certificado
sudo certbot --nginx -d unofficialbrasil.com.br -d www.unofficialbrasil.com.br

# 3. Seguir prompts:
# Email: seu-email@example.com
# Termos: (A)gree
# Redirect HTTP → HTTPS: Yes

# 4. Testar configuração
sudo nginx -t

# 5. Recarregar Nginx
sudo systemctl reload nginx

# 6. Verificar auto-renovação
sudo certbot renew --dry-run

# Certificado renova automaticamente 30 dias antes de expirar
```

#### 5. Secrets Audit

**Teste:**
```bash
# 1. Verificar .env não está no git
cat .gitignore | grep ".env"
# ESPERADO: .env aparece

# 2. Procurar secrets no código
grep -r "password\|api_key\|secret\|token\|bearer" \
  src/ app/ lib/ config/ \
  --exclude-dir=node_modules \
  --exclude="*.md" \
  --exclude="*.json" \
  | grep -v "// " \
  | grep -v "process.env"

# SE ENCONTRAR algo tipo:
# const apiKey = "sk_live_abc123..." → VULNERABILIDADE CRÍTICA
# DEVE SER: const apiKey = process.env.API_KEY

# 3. Verificar histórico do git
git log --all --oneline | head -20
git log --all -p | grep -E "BEGIN PRIVATE KEY|password.*=|api_key.*=" | head -10

# SE ENCONTRAR secrets em commits antigos:
# - Revogue o secret (regenere no serviço)
# - Remova do histórico (git filter-branch ou BFG Repo-Cleaner)
```

**Solução: Environment Variables**

```bash
# .env (NUNCA commitar no git)
DATABASE_URL="mysql://user:password@localhost:3306/unofficial"
REDIS_URL="redis://localhost:6379"
JWT_SECRET="gere-um-secret-aleatorio-aqui-32-chars-min"
WHATSAPP_API_TOKEN="seu-token-whatsapp-api"
INSTAGRAM_API_TOKEN="seu-token-instagram-api"
NEXT_PUBLIC_GA_ID="G-XXXXXXXXXX"
SENTRY_DSN="https://xxx@sentry.io/xxx"
```

```typescript
// lib/config.ts - Centralizar configs
export const config = {
  database: {
    url: process.env.DATABASE_URL,
  },
  redis: {
    url: process.env.REDIS_URL,
  },
  auth: {
    jwtSecret: process.env.JWT_SECRET,
    otpExpiry: 300, // 5 minutos
  },
  integrations: {
    whatsapp: {
      apiToken: process.env.WHATSAPP_API_TOKEN,
      apiUrl: 'https://graph.facebook.com/v18.0',
    },
    instagram: {
      apiToken: process.env.INSTAGRAM_API_TOKEN,
    },
  },
  monitoring: {
    sentryDsn: process.env.SENTRY_DSN,
  },
};

// Validar na inicialização
function validateConfig() {
  const required = [
    'DATABASE_URL',
    'REDIS_URL',
    'JWT_SECRET',
    'WHATSAPP_API_TOKEN',
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing env vars: ${missing.join(', ')}`);
  }
}

validateConfig();
```

**Deploy em produção:**
```bash
# SSH no servidor
ssh user@72.61.134.35

# Criar .env em produção
cd /path/to/unofficial-communities
nano .env

# Colar valores de produção (NÃO os mesmos de dev)
# Salvar: Ctrl+X, Y, Enter

# Restringir permissões
chmod 600 .env

# Reiniciar app
docker-compose down
docker-compose up -d

# Verificar carregou
docker logs unofficial-api | grep "Config loaded"
```

#### 6. Dependency Security

**Teste:**
```bash
# 1. Audit npm packages
cd /path/to/unofficial-communities
npm audit

# ESPERADO: 0 critical, 0 high
# SE HOUVER vulnerabilidades:

# 2. Fix automático
npm audit fix

# 3. Se fix automático não funcionar
npm audit fix --force  # Cuidado: pode quebrar coisas

# 4. Se ainda houver vulnerabilidades críticas
# Verificar manualmente:
npm audit --json > audit.json
cat audit.json | jq '.vulnerabilities'

# Atualizar package específico:
npm install package-name@latest

# 5. Outdated packages
npm outdated

# Atualizar:
npm update
```

**Automatizar com GitHub Actions:**

```yaml
# .github/workflows/security.yml
name: Security Audit

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 0 * * 1'  # Toda segunda-feira

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run npm audit
        run: npm audit --audit-level=moderate
        continue-on-error: true
      
      - name: Check for outdated packages
        run: npm outdated
        continue-on-error: true
```

---

## PARTE 2: SEO TÉCNICO

### 2.1 On-Page SEO

**Template para cada página:**

```tsx
// app/page.tsx (Homepage)
import type { Metadata } from 'next';

export const metadata: Metadata = {
  // Título (50-60 chars, inclui keyword principal)
  title: 'Gamificação para Comunidades WhatsApp | Unofficial Brasil',
  
  // Description (150-160 chars, inclui CTA)
  description: 'Aumente engajamento em 30% com gamificação ética. XP, missões, mundo 3D. Teste grátis 7 dias sem cartão.',
  
  // Canonical URL (evita duplicate content)
  alternates: {
    canonical: 'https://unofficialbrasil.com.br',
  },
  
  // Open Graph (Facebook, WhatsApp)
  openGraph: {
    type: 'website',
    title: 'Gamificação para Comunidades WhatsApp | Unofficial Brasil',
    description: 'Aumente engajamento em 30% com gamificação ética.',
    url: 'https://unofficialbrasil.com.br',
    siteName: 'Unofficial Brasil',
    images: [
      {
        url: 'https://unofficialbrasil.com.br/og-image.jpg',
        width: 1200,
        height: 630,
        alt: 'Unofficial Brasil - Gamificação ética para comunidades',
      },
    ],
    locale: 'pt_BR',
  },
  
  // Twitter Card
  twitter: {
    card: 'summary_large_image',
    title: 'Gamificação para Comunidades WhatsApp',
    description: 'Aumente engajamento em 30% com gamificação ética.',
    images: ['https://unofficialbrasil.com.br/og-image.jpg'],
  },
  
  // Robots (default: index, follow)
  robots: {
    index: true,
    follow: true,
    googleBot: {
      index: true,
      follow: true,
    },
  },
  
  // Verificação do Search Console (se usar meta tag)
  verification: {
    google: 'seu-codigo-aqui', // Opcional: do Google Search Console
  },
};

export default function HomePage() {
  return (
    <>
      {/* JSON-LD Structured Data */}
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{
          __html: JSON.stringify({
            '@context': 'https://schema.org',
            '@type': 'SoftwareApplication',
            name: 'Unofficial Brasil',
            description: 'Plataforma de gamificação para comunidades WhatsApp',
            url: 'https://unofficialbrasil.com.br',
            applicationCategory: 'BusinessApplication',
            operatingSystem: 'Web',
            offers: {
              '@type': 'Offer',
              price: '0',
              priceCurrency: 'BRL',
              description: '7 dias grátis',
            },
            aggregateRating: {
              '@type': 'AggregateRating',
              ratingValue: '4.8',
              ratingCount: '15',
            },
          }),
        }}
      />
      
      <main>
        {/* H1 - Apenas 1 por página */}
        <h1>Gamificação Ética para Comunidades WhatsApp</h1>
        
        {/* Resto do conteúdo */}
      </main>
    </>
  );
}
```

**Criar OG Image (1200×630px):**

```bash
# Usar Canva (grátis)
# 1. Criar design 1200×630px
# 2. Adicionar:
#    - Logo Unofficial Brasil
#    - Headline: "Gamificação Ética para Comunidades"
#    - Screenshot do produto (UC World ou dashboard)
#    - Cores: #134252 (primary), #E5E0DA (secondary)
# 3. Download como JPG (qualidade alta)
# 4. Otimizar:

# Instalar ImageMagick (se não tiver)
sudo apt install imagemagick

# Otimizar imagem
convert og-image-original.jpg \
  -quality 85 \
  -strip \
  og-image.jpg

# Verificar tamanho (deve ser < 300KB)
ls -lh og-image.jpg

# Upload para /public/
cp og-image.jpg /path/to/unofficial-communities/public/
```

### 2.2 Sitemap Dinâmico

```typescript
// app/sitemap.ts
import type { MetadataRoute } from 'next';

export default function sitemap(): MetadataRoute.Sitemap {
  const baseUrl = 'https://unofficialbrasil.com.br';
  
  // Páginas estáticas
  const staticPages = [
    '',
    '/precos',
    '/solucoes',
    '/solucoes/ecommerce',
    '/solucoes/infoprodutores',
    '/solucoes/agencias',
    '/recursos',
    '/seguranca',
    '/sobre',
    '/contato',
    '/privacidade',
    '/termos',
    '/cookies',
    '/reembolso',
  ];
  
  const staticUrls: MetadataRoute.Sitemap = staticPages.map(page => ({
    url: `${baseUrl}${page}`,
    lastModified: new Date(),
    changeFrequency: page === '' ? 'weekly' : 'monthly',
    priority: page === '' ? 1.0 : 0.8,
  }));
  
  // Blog posts (buscar do database ou filesystem)
  const blogPosts = await getBlogPosts(); // Implementar função
  const blogUrls: MetadataRoute.Sitemap = blogPosts.map(post => ({
    url: `${baseUrl}/blog/${post.slug}`,
    lastModified: new Date(post.updatedAt),
    changeFrequency: 'monthly',
    priority: 0.7,
  }));
  
  return [...staticUrls, ...blogUrls];
}
```

### 2.3 robots.txt

```typescript
// app/robots.ts
import type { MetadataRoute } from 'next';

export default function robots(): MetadataRoute.Robots {
  return {
    rules: [
      {
        userAgent: '*',
        allow: '/',
        disallow: [
          '/admin',
          '/api',
          '/webhooks',
          '/painel',
          '/_next',
          '/static',
        ],
      },
      {
        userAgent: 'GPTBot', // Block ChatGPT crawler (opcional)
        disallow: ['/'],
      },
    ],
    sitemap: 'https://unofficialbrasil.com.br/sitemap.xml',
  };
}
```

### 2.4 Google Search Console Setup

```bash
# 1. Acesse: https://search.google.com/search-console

# 2. Add property: unofficialbrasil.com.br

# 3. Verificação - Método 1 (HTML file):
#    - Download: googleXXXXXX.html
#    - Upload para: /public/googleXXXXXX.html
#    - Clique "Verify"

# 4. Ou Método 2 (Meta tag):
#    - Copie tag: <meta name="google-site-verification" content="...">
#    - Adicione em metadata (ver exemplo acima)
#    - Clique "Verify"

# 5. Submeter sitemap:
#    - Sidebar: Sitemaps
#    - Adicionar: https://unofficialbrasil.com.br/sitemap.xml
#    - Enviar

# 6. Configurar propriedades:
#    - Domain: unofficialbrasil.com.br
#    - País: Brasil
#    - Moeda: BRL

# 7. Aguardar indexação (3-7 dias)
```

### 2.5 Google Analytics 4 Setup

```typescript
// app/layout.tsx
import { GoogleAnalytics } from '@next/third-parties/google';

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="pt-BR">
      <body>
        {children}
        
        {/* Google Analytics 4 */}
        <GoogleAnalytics gaId={process.env.NEXT_PUBLIC_GA_ID} />
      </body>
    </html>
  );
}
```

```bash
# 1. Criar conta GA4: https://analytics.google.com

# 2. Criar propriedade:
#    - Nome: Unofficial Brasil
#    - Timezone: (GMT-03:00) São Paulo
#    - Moeda: Brazilian Real (R$)

# 3. Criar Data Stream:
#    - Plataforma: Web
#    - URL: https://unofficialbrasil.com.br
#    - Stream name: Website

# 4. Copiar Measurement ID (formato: G-XXXXXXXXXX)

# 5. Adicionar ao .env:
NEXT_PUBLIC_GA_ID="G-XXXXXXXXXX"

# 6. Deploy e testar:
#    - Abra site
#    - GA4 Realtime: Deve aparecer 1 usuário ativo
```

### 2.6 Keywords & Content Strategy

**Keywords Primárias (Brasil):**

| Keyword | Volume/mês | Dificuldade | Página Alvo |
|---------|------------|-------------|-------------|
| gamificação whatsapp | 320 | Média | /home, /blog/gamificacao |
| retenção membros comunidade | 210 | Média | /solucoes, /blog/retencao |
| plataforma comunidade brasil | 150 | Alta | /home |
| whatsapp grupo engajamento | 180 | Média | /solucoes/agencias |
| monetização comunidade online | 95 | Alta | /blog |
| living graph comunidades | 30 | Baixa | /blog/living-graph |

**Como usar:**
1. **Title tag:** Inclua keyword principal
2. **H1:** Keyword + benefício
3. **Corpo:** 2-3 menções naturais (densidade 1-2%)
4. **URL:** /blog/gamificacao-whatsapp (keyword no slug)
5. **Alt text:** Imagens relacionadas

**Exemplo:**
```markdown
---
title: "Gamificação para WhatsApp: Guia Completo 2026"
slug: gamificacao-whatsapp
---

# Gamificação para WhatsApp: Como Aumentar Engajamento em Comunidades

Gamificação para WhatsApp é a estratégia #1 para aumentar engajamento...

[Texto continua com 2-3 menções de "gamificação whatsapp"]

![Interface de gamificação do Unofficial Brasil](image.jpg)
Alt text: "Dashboard de gamificação para WhatsApp mostrando XP e missões"
```

---

## PARTE 3: PERFORMANCE OPTIMIZATION

### 3.1 Core Web Vitals

**Targets:**
- **LCP (Largest Contentful Paint):** < 2.5s
- **FID (First Input Delay):** < 100ms
- **CLS (Cumulative Layout Shift):** < 0.1

**Testar:**
```bash
# 1. PageSpeed Insights
# Abra: https://pagespeed.web.dev/
# Teste: https://unofficialbrasil.com.br

# 2. Lighthouse (local)
npm install -g lighthouse
lighthouse https://unofficialbrasil.com.br --view

# 3. WebPageTest
# Abra: https://www.webpagetest.org/
# Teste: unofficialbrasil.com.br
# Location: São Paulo, Brazil
```

**Otimizar LCP:**

```typescript
// next.config.js
const nextConfig = {
  // 1. Imagem optimization
  images: {
    formats: ['image/webp', 'image/avif'],
    deviceSizes: [640, 750, 828, 1080, 1200, 1920],
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
  },
  
  // 2. Compression
  compress: true,
  
  // 3. Experimental features
  experimental: {
    optimizeCss: true,
  },
};

module.exports = nextConfig;
```

```tsx
// Usar Next/Image para LCP image
import Image from 'next/image';

export function Hero() {
  return (
    <section>
      <h1>Gamificação para Comunidades</h1>
      
      {/* priority = preload esta imagem */}
      <Image
        src="/hero-screenshot.jpg"
        alt="Interface do Unofficial Brasil"
        width={1200}
        height={630}
        priority // ← IMPORTANTE para LCP image
        placeholder="blur"
        blurDataURL="data:image/jpeg;base64,..."
      />
    </section>
  );
}
```

**Otimizar imagens:**

```bash
# Instalar sharp (Next.js usa automaticamente)
npm install sharp

# Comprimir manualmente (se necessário)
# Instalar cwebp
sudo apt install webp

# Converter JPG → WebP
cwebp -q 80 hero-screenshot.jpg -o hero-screenshot.webp

# Verificar tamanho
ls -lh hero-screenshot.*
# hero-screenshot.jpg: 250KB
# hero-screenshot.webp: 85KB (66% menor)
```

**Otimizar FID:**

```tsx
// 1. Usar dynamic import para JS pesado
import dynamic from 'next/dynamic';

const UCWorldViewer = dynamic(() => import('@/components/UCWorldViewer'), {
  ssr: false,
  loading: () => <p>Carregando mundo 3D...</p>,
});

// 2. Defer non-critical scripts
<script src="/analytics.js" defer />

// 3. Use Web Workers para processamento pesado
// worker.ts
self.onmessage = (event) => {
  const result = heavyComputation(event.data);
  self.postMessage(result);
};

// component.tsx
const worker = new Worker('/worker.js');
worker.postMessage(data);
worker.onmessage = (event) => {
  console.log(event.data);
};
```

**Otimizar CLS:**

```css
/* 1. Reserve espaço para imagens */
.hero-image {
  aspect-ratio: 16 / 9;
  width: 100%;
}

/* 2. Use min-height para dynamic content */
.testimonials {
  min-height: 400px;
}

/* 3. Evite inserir conteúdo acima de existing content */
/* BAD: Banner no topo empurra conteúdo */
/* GOOD: Banner sticky ou no bottom */
.banner {
  position: sticky;
  bottom: 0;
}
```

### 3.2 Caching Strategy

**Browser Caching (Nginx):**

```nginx
# Cache static assets (images, CSS, JS, fonts)
location ~* \.(jpg|jpeg|png|gif|ico|svg|webp|css|js|woff|woff2|ttf|otf)$ {
    expires 1y;
    add_header Cache-Control "public, immutable";
}

# Cache HTML (shorter duration)
location ~* \.(html)$ {
    expires 1h;
    add_header Cache-Control "public, must-revalidate";
}

# No cache for API
location /api {
    expires off;
    add_header Cache-Control "no-store, no-cache, must-revalidate, proxy-revalidate";
}
```

**Redis Caching (API):**

```typescript
// lib/cache.ts
import Redis from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

export async function getCached<T>(
  key: string,
  fetcher: () => Promise<T>,
  ttl: number = 300 // 5 minutos default
): Promise<T> {
  // Try cache first
  const cached = await redis.get(key);
  if (cached) {
    return JSON.parse(cached);
  }
  
  // Cache miss: fetch data
  const data = await fetcher();
  
  // Store in cache
  await redis.setex(key, ttl, JSON.stringify(data));
  
  return data;
}

// Uso
export async function GET(req: Request) {
  const communityId = 123;
  
  const health = await getCached(
    `community:${communityId}:health`,
    async () => {
      // Expensive computation
      return await computeCommunityHealth(communityId);
    },
    300 // Cache por 5 minutos
  );
  
  return Response.json(health);
}
```

---

## PARTE 4: MONITORING & ALERTING

### 4.1 Sentry (Error Tracking)

```bash
# 1. Criar conta: https://sentry.io (free tier: 5k errors/month)

# 2. Criar projeto:
#    - Platform: Next.js
#    - Nome: Unofficial Brasil

# 3. Copiar DSN

# 4. Instalar SDK
npm install @sentry/nextjs

# 5. Init Sentry
npx @sentry/wizard@latest -i nextjs
```

```typescript
// sentry.client.config.ts
import * as Sentry from '@sentry/nextjs';

Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  
  // Performance monitoring
  tracesSampleRate: 0.1, // 10% das transações
  
  // Session replay
  replaysSessionSampleRate: 0.1,
  replaysOnErrorSampleRate: 1.0, // 100% quando há erro
  
  // Environment
  environment: process.env.NODE_ENV,
  
  // Release tracking
  release: process.env.VERCEL_GIT_COMMIT_SHA,
});
```

**Usar Sentry:**

```typescript
// Capturar erro manualmente
try {
  await riskyOperation();
} catch (error) {
  Sentry.captureException(error, {
    tags: { section: 'payment' },
    extra: { userId: user.id },
  });
  throw error;
}

// Breadcrumbs (tracking de ações antes do erro)
Sentry.addBreadcrumb({
  category: 'auth',
  message: 'User attempted login',
  level: 'info',
});
```

### 4.2 Uptime Monitoring

```bash
# Opção 1: UptimeRobot (grátis, 50 monitors)
# 1. Criar conta: https://uptimerobot.com
# 2. Add New Monitor:
#    - Type: HTTPS
#    - URL: https://unofficialbrasil.com.br
#    - Interval: 5 minutes
#    - Alert: Email quando down
# 3. Adicionar monitor para API:
#    - URL: https://unofficialbrasil.com.br/api/health

# Opção 2: Status page público (Statuspage.io free tier)
# https://www.atlassian.com/software/statuspage
```

**Criar API /health:**

```typescript
// app/api/health/route.ts
import { db } from '@/lib/db';
import { redis } from '@/lib/redis';

export async function GET() {
  const checks = {
    timestamp: new Date().toISOString(),
    status: 'healthy',
    checks: {},
  };
  
  // Check database
  try {
    await db.$queryRaw`SELECT 1`;
    checks.checks.database = 'ok';
  } catch (error) {
    checks.status = 'unhealthy';
    checks.checks.database = 'error';
  }
  
  // Check Redis
  try {
    await redis.ping();
    checks.checks.redis = 'ok';
  } catch (error) {
    checks.status = 'unhealthy';
    checks.checks.redis = 'error';
  }
  
  const statusCode = checks.status === 'healthy' ? 200 : 503;
  
  return Response.json(checks, { status: statusCode });
}
```

### 4.3 Custom Metrics Dashboard

```typescript
// lib/metrics.ts
import { redis } from './redis';

export async function trackMetric(
  metric: string,
  value: number,
  tags?: Record<string, string>
) {
  const key = `metrics:${metric}:${Date.now()}`;
  await redis.setex(key, 86400, JSON.stringify({ value, tags })); // 24h TTL
}

// Uso
await trackMetric('signup', 1, { source: 'organic' });
await trackMetric('api_response_time', 145, { endpoint: '/api/auth/login' });
```

**Dashboard simples (Google Sheets + Apps Script):**

```javascript
// Google Apps Script
function fetchMetrics() {
  const response = UrlFetchApp.fetch('https://unofficialbrasil.com.br/api/metrics');
  const data = JSON.parse(response.getContentText());
  
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('Metrics');
  sheet.appendRow([
    new Date(),
    data.signups,
    data.activeUsers,
    data.errorRate,
  ]);
}

// Trigger: Run hourly
```

---

## CHECKLIST FINAL

**Segurança**
- [ ] OWASP Top 10 checklist completo
- [ ] SSL A+ rating (SSLLabs)
- [ ] Security headers A rating (SecurityHeaders.com)
- [ ] Rate limiting ativo
- [ ] Secrets não expostos (código + git history)
- [ ] npm audit: 0 critical/high
- [ ] Backups automáticos + restoration testada

**SEO**
- [ ] Meta tags em todas páginas (title, description)
- [ ] Sitemap.xml gerado + submetido GSC
- [ ] robots.txt configurado
- [ ] Structured data (Schema.org)
- [ ] Google Search Console verificado
- [ ] Google Analytics 4 instalado
- [ ] OG image criada (1200×630px)
- [ ] Alt text em todas imagens
- [ ] Internal linking (3-5 links por página)

**Performance**
- [ ] Lighthouse score > 90
- [ ] Core Web Vitals: All green
  - LCP < 2.5s
  - FID < 100ms
  - CLS < 0.1
- [ ] Imagens otimizadas (WebP, < 100KB)
- [ ] Lazy loading configurado
- [ ] Gzip compression ativo
- [ ] Browser caching configurado

**Monitoring**
- [ ] Sentry instalado + testado
- [ ] Uptime monitor configurado (UptimeRobot)
- [ ] /api/health endpoint funcional
- [ ] Status page público (opcional)
- [ ] Alertas configurados (email on downtime/errors)

**Deploy**
- [ ] .env configurado em produção
- [ ] Nginx configurado + testado
- [ ] SSL certificate válido
- [ ] DNS apontando corretamente
- [ ] Docker containers rodando
- [ ] Database migrations aplicadas
- [ ] Rollback procedure testado

**Se TUDO marcado → PRONTO PARA LANÇAR 🚀**
