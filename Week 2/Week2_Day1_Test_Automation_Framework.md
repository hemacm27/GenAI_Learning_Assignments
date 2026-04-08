# Enterprise Test Automation Framework (Playwright + TypeScript)

## 🛠 Tech Stack
- **Language:** TypeScript
- **Engine:** Playwright
- **Pattern:** Screenplay Pattern
- **CI/CD:** GitHub Actions
- **Data:** JSON/CSV
- **Reporting:** Playwright HTML Report

## 🏗 Key Components

### 1. Screenplay Pattern Implementation
- **Tasks:** Located in `src/tasks/`. These represent "What" the user does.
- **Actions:** Located in `src/actions/`. These are the "How" (wrappers around Playwright's native commands).
- **Questions:** Located in `src/questions/`. Used for assertions to verify the UI state.

playwright-enterprise-framework/
├── .github/
│   └── workflows/
│       └── main.yml              # GitHub Actions CI pipeline
├── src/
│   ├── actors/                   # Actor definitions
│   ├── tasks/                    # High-level business tasks (Login, Checkout)
│   ├── actions/                  # Low-level atomic interactions
│   ├── questions/                # Assertions and state queries
│   ├── ui/                       # Locators/Selectors (Component-based)
│   └── utils/
│       ├── csv-reader.ts         # Utility to parse CSV files
│       └── env-config.ts         # Environment variable loader
├── test-data/
│   ├── staging/
│   │   ├── users.json
│   │   └── products.csv
│   └── prod/
│       └── users.json
├── tests/                        # Actual test specs
│   ├── auth/
│   └── orders/
├── .env                          # Local secrets (gitignored)
├── .env.staging                  # Staging environment variables
├── playwright.config.ts          # Global Playwright configuration
├── package.json
└── tsconfig.json

### 2. Test Data Management
Test data is separated by environment in the `test-data/` directory. Use the `csv-reader.ts` utility to inject data into tests dynamically.

### 3. CI/CD Pipeline
The framework is configured for GitHub Actions.
- **Parallelism:** Controlled via `workers` in `playwright.config.ts`.
- **Artifacts:** HTML reports and trace files are uploaded on failure.

## 🏃 How to Run
1. Install dependencies: `npm install`
2. Run tests: `npx playwright test`
3. View Report: `npx playwright show-report`