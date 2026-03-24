# 🌐 Multi-Domain Store Setup

Documentation des changements pour le support multi-domaine des stores.

## Changements Effectués (2026-01-13)

### Objectif
Permettre à chaque store d'avoir son propre domaine personnalisé via CloudFront Lambda@Edge.

### Store par Défaut
- **Ancien**: `yunicbrightvision`
- **Nouveau**: `default-store` (configurable via `environment.defaultStore`)

---

## Configuration Environment

### `src/environments/environment.ts` (Development)
```typescript
export const environment = {
  production: false,
  baseUrl: 'https://3decommbackend-production-8a9a.up.railway.app/api/v1/',
  storeUrl: 'https://dawky10en11i3.cloudfront.net',
  defaultStore: 'default-store'  // ← Nouveau
};
```

### `src/environments/environment.prod.ts` (Production)
```typescript
export const environment = {
  production: true,
  baseUrl: 'https://3decommbackend-production-8a9a.up.railway.app/api/v1/',
  storeUrl: 'https://dawky10en11i3.cloudfront.net',
  defaultStore: 'default-store'  // ← Nouveau
};
```

---

## Fichiers Modifiés

### 1. `src/app/app-routing.module.ts` (ligne 32)

```diff
- data: { slug: 'yunicbrightvision' },
+ data: { slug: 'default-store' },
```

---

### 2. `src/app/store/single-store-banner/single-store-banner.component.ts` (ligne 59)

```diff
- this.store_slug = this.getSlugFromCookie() || 'yunicbrightvision';
+ this.store_slug = this.getSlugFromCookie() || environment.defaultStore;
```

---

### 3. `src/app/pages/search/search.component.ts`

**Import ajouté (ligne 5):**
```typescript
import { environment } from 'src/environments/environment';
```

**Lignes 32, 109:**
```diff
- "store_slug": this.store_slug ? this.store_slug : 'yunicbrightvision'
+ "store_slug": this.store_slug ? this.store_slug : environment.defaultStore
```

---

### 4. `src/app/pages/sitemap/sitemap.component.html` (lignes 38-76)

10 liens mis à jour de `yunicbrightvision` → `default-store`.

---

## Architecture Lambda@Edge

```
Custom Domain (techaven.com)
       ↓
CloudFront + Lambda@Edge
       ↓
Set Cookie: storeslug=tech-haven
       ↓
Angular App lit le cookie → charge le bon store
```

### Flux de Résolution du Store Slug

1. **URL Param** `/vendor/:slug` → utilise le slug de l'URL
2. **Cookie** `storeslug` (posé par Lambda@Edge) → utilise la valeur du cookie
3. **Fallback** → `environment.defaultStore`

---

## Pour Changer le Store par Défaut

Modifier uniquement les fichiers environment:
```typescript
defaultStore: 'mon-nouveau-store'
```

Aucun autre fichier à modifier !

---

## Prochaines Étapes

- [ ] Créer Lambda@Edge functions (Viewer Request + Response)
- [ ] Configurer CloudFront CNAMEs pour custom domains
- [ ] Créer certificat SSL multi-domaine dans ACM
- [ ] Configurer DNS pour chaque domaine custom
