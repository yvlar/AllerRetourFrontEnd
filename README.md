# AllerRetour — Frontend

Interface React du projet **CoRoute** — covoiturage domicile-travail en région québécoise.  
Cours **GLO-2004** · Université Laval.

---

## Stack technique

| Catégorie | Outil |
|---|---|
| UI | React 18 + Vite 5 |
| Style | CSS variables globales |
| Client API | `fetch` natif (src/shared/services/api.js) |
| Auth | JWT Bearer — `useAuth` hook (mémoire) |
| Linting | ESLint 9 (react, hooks, a11y, security) |
| Formatage | Prettier |
| Tests | Vitest + React Testing Library |
| Couverture | V8 (seuil : 70%) |
| Git hooks | Husky + lint-staged |
| Conteneur | Docker multi-stage (builder → nginx) |
| CI | GitHub Actions |

---

## Endpoints backend (CoRoute JAX-RS — base `:8080`)

| Méthode | Chemin | Auth | Description |
|---|---|---|---|
| `POST` | `/utilisateurs/inscription` | — | Créer un compte |
| `POST` | `/utilisateurs/connexion` | — | Connexion → JWT |
| `GET` | `/trajets/match?depart=&destination=&jours=LUNDI` | — | Recherche matching |
| `GET` | `/trajets?depart=&destination=&date=` | — | Liste simple |
| `GET` | `/trajets/{id}` | — | Détail trajet |
| `POST` | `/trajets` | ✅ Bearer | Créer trajet |
| `DELETE` | `/trajets/{id}` | ✅ Bearer | Supprimer trajet |
| `POST` | `/trajets/{id}/reservations` | ✅ Bearer | Réserver |
| `DELETE` | `/trajets/{id}/reservations/{resId}` | ✅ Bearer | Annuler |
| `GET` | `/trajets/{id}/reservations` | ✅ Bearer | Liste réservations |

**Enums `JourSemaine`** : `LUNDI MARDI MERCREDI JEUDI VENDREDI SAMEDI DIMANCHE`  
**Enums `TrajetType`** : `PONCTUEL REGULIER`  
**Heure** : format ISO `"07:00:00"` → affiché `"07h00"` via `formatHeure()`

---

## Structure du projet

```
src/
├── features/
│   ├── auth/
│   │   └── components/AuthModal.jsx    # onLogin / onRegister (appels API)
│   ├── trajets/
│   │   ├── components/
│   │   │   ├── TrajetCard.jsx          # Utilise placesRestantes, prixParPassager
│   │   │   └── ResultsSection.jsx      # Gère loading / error
│   │   ├── data/mockTrajets.js         # Aligné sur MatchingResponse
│   │   └── utils/
│   │       ├── trajetStatus.js         # getTrajetStatus(placesRestantes)
│   │       └── formatters.js           # formatHeure("07:00:00") → "07h00"
│   └── routes/
│       ├── components/RouteCard.jsx
│       └── data/mockRoutes.js
├── shared/
│   ├── services/api.js                 # Client fetch centralisé
│   ├── hooks/
│   │   ├── useAuth.js                  # login / register / logout + JWT
│   │   ├── useSearch.js                # matchTrajets + conversion enums
│   │   └── useToast.js
│   ├── constants/
│   │   ├── jours.js                    # JOUR_LABEL_TO_ENUM + labelsToEnums()
│   │   └── colors.js
│   └── components/  Navbar / Hero / Toast / JoursPicker
tests/
├── unit/    trajetStatus · formatters · jours · useToast · useSearch · useAuth
└── integration/    TrajetCard · AuthModal · Toast · api
```

---

## Proxy API — comment ça marche

```
DEV (Vite)   : /api/trajets/match → vite proxy rewrite → http://localhost:8080/trajets/match
PROD (nginx) : /api/trajets/match → nginx rewrite      → http://api:8080/trajets/match
```

La variable `VITE_API_URL=/api` est commune aux deux environnements.

---

## Démarrage rapide

```bash
cp .env.example .env
npm install
npm run dev           # http://localhost:5173
```

### Docker Compose complet

```bash
# Structure attendue :
# workspace/
# ├── coroute-backend/     ← projet Maven (avec son Dockerfile)
# └── allerretour-frontend/

docker compose up --build

# → App :           http://localhost
# → API :           http://localhost:8080
# → Mongo Express : http://localhost:8081
```

### Mode dev avec hot-reload

```bash
docker compose -f docker-compose.yml -f docker-compose.dev.yml up --build
# → Vite HMR : http://localhost:5173
```

---

## Commandes

```bash
npm run lint          # ESLint 0 warning
npm run format:check  # Prettier check
npm run test          # Tous les tests
npm run test:coverage # Rapport couverture (seuil 70%)
npm run build         # Build production
npm run audit:security
```

---

## Git Hooks (Husky)

| Hook | Action |
|---|---|
| `pre-commit` | lint-staged → ESLint + Prettier sur les fichiers modifiés |
| `pre-push` | `npm run test` |

---

**CoRoute · GLO-2004 · Université Laval**
