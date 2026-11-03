# RecoMind: Graph-Based Movie Recommendation System
**Academic Project for B.Tech Data Structures & Algorithms Curriculum**

RecoMind is a complete full-stack movie recommendation and discovery system that combines a textbook **C++17 graph recommendation engine** with an enterprise-hardened **Node.js/Express backend (node:sqlite)** and a **vanilla HTML/CSS/JS frontend** with strict **Role-Based Access Control (RBAC)**.

---

## 1. System Architecture

```mermaid
graph TD
    Client[Vanilla HTML5/CSS3/ES6 Client]
    
    subgraph Frontend Pages
        L1[login.html - User Portal]
        L2[admin-login.html - Admin Command Access]
        A1[account.html - User Dashboard 'My space']
        A2[admin.html - Administrator Control Console]
        D1[discover.html / title.html / index.html]
        Lab[lab.html - DSA Traversal Visualizer]
    end

    subgraph Express Backend Layer (Node.js 20+)
        Security[Security Middleware: CSP, HSTS, X-Frame-Options, CSRF]
        RateLimit[Composite RateLimiter: 5 attempts/15 min]
        AuthGuard[RBAC Middleware: requireAuth & requireRole('admin')]
        IPC[Subprocess Bridge child_process.execFile]
    end

    subgraph Native Engine Layer (C++17)
        Graph[Bipartite Adjacency List Graph]
        BFS[3-Hop Multi-Hop BFS Collaborative Filtering]
        Scorer[Multi-Factor Composite Scorer & Jaccard Index]
        Heap[Array-Based Binary Max Heap Top-K Ranking]
    end

    subgraph Persistence Layer (SQLite node:sqlite)
        DB[(recomind.db)]
        Users[users & sessions: scrypt hashes, fresh DB role validation]
        Audit[admin_audit_log & login_attempts: Immutable trails]
        Data[movies, ratings, watch_history, title_interactions]
    end

    Client --> FrontendPages
    FrontendPages --> Security
    Security --> RateLimit
    RateLimit --> AuthGuard
    AuthGuard --> DB
    AuthGuard --> IPC
    IPC --> Graph
    Graph --> BFS
    BFS --> Scorer
    Scorer --> Heap
    Heap --> IPC
```

---

## 2. Key Role-Based Capabilities (USER vs ADMIN)

The platform enforces two strictly separated operational roles with dedicated authentication flows and permissions:

### 👤 Standard User Role (`USER`)
- **Login Portal:** Dedicated, cinematic `login.html` with instant sign-in / registration toggle.
- **Personal Cinema Dashboard (`account.html`):**
  - Profile identity card with member since date, role badge `[USER]`, and dynamic recommendation graph node (e.g. `U26`).
  - Change display name and change password (with session invalidation across other devices).
  - Choice of 5 cinematic themes persisted to database (`midnight`, `ocean`, `rose`, `ember`, `ivory`).
  - Personal activity metrics: ratings given, saved watchlist, favourites, reviews written.
  - Personal C++ graph recommendations generated dynamically for the user's specific graph node.
  - Interactive management for ratings (edit/delete), watchlist (remove), favourites (remove), and reviews (delete).
  - Sign out of all devices button.
- **Server Data Isolation:** All reads and writes scope strictly to the verified session identity; users cannot access or tamper with other users' data (IDOR protected).

### 🛡️ Administrator Role (`ADMIN`)
- **Login Portal:** Dedicated command interface `admin-login.html` with access warning and audit tracking. Calls `/api/admin/login` (generic 401 on non-admin).
- **Control Console (`admin.html`):**
  - **Overview Cards:** Live counts of total users, active accounts, administrators, catalogue movies, graph rating edges, and community reviews.
  - **User Account Governance:** Searchable, paginated user management table.
    * Deactivate / reactivate accounts (immediately revokes active sessions).
    * Promote to Admin / Demote to User.
    * Force sign-out (purges active sessions).
    * Safety Rules: Admin cannot deactivate or demote themselves; last remaining active admin cannot be deactivated or demoted.
  - **Content Moderation:** Table of community reviews with author info, rating, and date, plus "Delete Review" button.
  - **Immutable Security Audit Log:** Tamper-evident records tracking timestamp, admin identity, action, target user, detailed JSON change snapshot, and IP address.
  - **Dataset Browser & Poster Sync:** Inspect raw SQLite tables and execute parallel TMDB poster syncing.
  - **Broadcast Announcements:** Post studio updates visible on all visitor dashboards.

---

## 3. Five Premier Cinematic Themes

RecoMind features 5 rich, hand-crafted themes with modern glassmorphism, glowing accents, and responsive typography:

1. **Midnight (Default):** Deep dark cinema palette (`#0a0e17`) with slate cards, vibrant violet/cyan glow.
2. **Ocean:** Deep blue/navy command palette (`#071426`) with cyan accents.
3. **Rose:** Warm dark plum aesthetic (`#160b12`) with warm rose and burgundy accents.
4. **Ember:** Cinematic warm espresso (`#140d07`) with amber and copper glow.
5. **Ivory:** Modern, crisp light aesthetic (`#f8fafc`) with indigo highlights.

Theme selection is instantly previewed, saved to `localStorage`, and persisted to the user's database record upon login.

---

## 4. Quickstart Guide (Single-Click Launcher)

### Prerequisites
- **Operating System:** Windows 10/11
- **Node.js:** v20.0.0 or higher
- **C++ Compiler (Optional):** MinGW GCC 8+ with C++17 support (pre-compiled binary already included).

### Launching the Application
Simply double-click `run.bat` in the project root, or execute from terminal:

```cmd
.\run.bat
```

What `run.bat` does automatically:
1. Verifies Node.js installation.
2. Checks for `engine/bin/recomind_engine.exe` (compiles via `g++` if missing).
3. Installs Node.js dependencies (`npm install` in `server/` if `node_modules` missing).
4. Initializes `server/.env` from `server/.env.example` if absent.
5. Launches Express server on `http://localhost:3000`.
6. Waits for the `/api/health` check and opens your default browser.

---

## 5. Automated Verification & Testing

### Backend & RBAC Integration Tests (127 Tests)
Run the full integration test suite:

```powershell
cd server
npm test
```
All **127 / 127 tests pass**, verifying:
- Health checks and anonymous route guards (HTTP 401).
- Catalogue filters and cold-start detection.
- Subprocess invocation of the native C++ recommendation engine binary.
- User data isolation (IDOR protection).
- Dual login segregation (`POST /api/auth/login` vs `POST /api/admin/login`).
- Fresh database role re-verification (`requireRole('admin')`).
- Rate limiting (HTTP 429 after 5 failed attempts with Retry-After header).
- Safety invariant blocks (self-deactivation and self-demotion protection).
- Session invalidation on password change and signout-all.

### C++ Engine Unit Tests (9 Tests)
Run the C++ unit test runner:

```powershell
cd engine
.\bin\test_engine.exe
```
All **9 / 9 unit tests pass**, validating:
- Bipartite adjacency list graph representation.
- Hash map index lookups and visited set cycle prevention.
- Multi-hop BFS collaborative search.
- Recursive DFS component traversal.
- Binary max-heap ordering and Top-K extraction.
- Watched history exclusion ($O(1)$).
- Composite scoring formula sensitivity.
- Cold-start fallback behavior.

---

## 6. API Reference Summary

### Authentication & User Space
| Method | Endpoint | Protection | Description |
| :--- | :--- | :--- | :--- |
| `POST` | `/api/auth/register` | Public | Registers a new user with role `user`. |
| `POST` | `/api/auth/login` | Rate Limited | Authenticates standard user; rejects admins (403). |
| `POST` | `/api/admin/login` | Rate Limited | Authenticates admin; rejects non-admins (401). |
| `GET` | `/api/auth/me` | Public | Returns current authenticated profile or 401. |
| `POST` | `/api/auth/logout` | Session | Revokes current session cookie. |
| `POST` | `/api/auth/signout-all` | `requireAuth` | Revokes all active sessions for current user. |
| `POST` | `/api/auth/change-password` | `requireAuth` | Updates password and invalidates other device sessions. |
| `GET` | `/api/account/data` | `requireAuth` | Returns personal watchlist, favourites, ratings, and reviews. |
| `PATCH` | `/api/account/profile` | `requireAuth` | Updates user's display name. |
| `PUT` | `/api/account/theme` | `requireAuth` | Persists chosen colour theme in database. |

### Title Interactions & Recommendations
| Method | Endpoint | Protection | Description |
| :--- | :--- | :--- | :--- |
| `GET` | `/api/titles/:source/:mediaType/:titleId/feedback` | Public | Returns average score, counts, comments, and personal rating. |
| `PUT` | `/api/titles/:source/:mediaType/:titleId/rating` | `requireAuth` | Saves half-star rating (0.5 to 5.0) and syncs to graph engine. |
| `DELETE` | `/api/titles/:source/:mediaType/:titleId/rating` | `requireAuth` | Deletes user's rating. |
| `PUT` | `/api/titles/:source/:mediaType/:titleId/favorite` | `requireAuth` | Toggles title favorite state. |
| `POST` | `/api/titles/:source/:mediaType/:titleId/comments` | `requireAuth` | Publishes community review comment. |
| `DELETE` | `/api/titles/:source/:mediaType/:titleId/comments` | `requireAuth` | Deletes user's own review comment. |
| `GET` | `/api/recommend/:userId` | Public/User | Executes native C++ binary and returns Top-K recommendations. |

### Administrator Governance
| Method | Endpoint | Protection | Description |
| :--- | :--- | :--- | :--- |
| `GET` | `/api/admin/users` | `requireRole('admin')` | Paginated and searchable user management list. |
| `PATCH` | `/api/admin/users/:id/status` | `requireRole('admin')` | Deactivates/reactivates user (kills active sessions). |
| `PATCH` | `/api/admin/users/:id/role` | `requireRole('admin')` | Promotes/demotes role (`user` vs `admin`). |
| `POST` | `/api/admin/users/:id/signout` | `requireRole('admin')` | Force terminates all active sessions for a user. |
| `DELETE` | `/api/admin/reviews/:id` | `requireRole('admin')` | Moderates and removes inappropriate reviews. |
| `GET` | `/api/admin/audit-logs` | `requireRole('admin')` | Paginated immutable security audit trail. |
| `POST` | `/api/admin/catalogue/sync-posters` | `requireRole('admin')` | Parallel TMDB poster synchronizer. |

---

## 7. Viva & Academic Documentation

Detailed technical proofs and exam prep materials are available in the `docs/` directory:
- [AUTH_AND_ROLES.md](docs/AUTH_AND_ROLES.md): Complete threat model, session design, and 5-minute viva demo script.
- [VIVA_QUESTIONS.md](docs/VIVA_QUESTIONS.md): 29 targeted viva Q&A covering graph data structures, algorithms, and security architecture.
- [COMPLEXITY_ANALYSIS.md](docs/COMPLEXITY_ANALYSIS.md): Rigorous Big-O time and space proofs for BFS, DFS, and Binary Max-Heap.
- [ALGORITHM_PSEUDOCODE.md](docs/ALGORITHM_PSEUDOCODE.md): Formatted textbook pseudocode.
- [ARCHITECTURE.md](docs/ARCHITECTURE.md): Full architectural overview.
#   d s - p r o j e c t  
 