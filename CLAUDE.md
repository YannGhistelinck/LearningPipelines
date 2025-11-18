# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a React application bootstrapped with Create React App (CRA). It uses React 19.2.0 and includes testing libraries configured with Jest and React Testing Library.

## Development Commands

### Running the application
```bash
npm start
```
Starts the development server at http://localhost:3000 with hot reloading enabled.

### Testing
```bash
npm test
```
Launches Jest test runner in interactive watch mode.

To run tests without watch mode:
```bash
npm test -- --watchAll=false
```

To run a specific test file:
```bash
npm test -- path/to/test-file.test.js
```

### Building
```bash
npm run build
```
Creates an optimized production build in the `build/` directory.

## Project Structure

```
src/
├── App.js              # Main application component
├── App.css             # Application styles
├── App.test.js         # Application tests
├── index.js            # React entry point with ReactDOM.createRoot
├── index.css           # Global styles
├── setupTests.js       # Jest/RTL test configuration
└── reportWebVitals.js  # Performance monitoring utilities

public/
├── index.html          # HTML template
├── manifest.json       # PWA manifest
└── [static assets]     # Favicons, logos, robots.txt
```

## Architecture Notes

- **React 19**: Uses the modern `ReactDOM.createRoot` API (not legacy `ReactDOM.render`)
- **Strict Mode**: Components are wrapped in `<React.StrictMode>` for development warnings
- **Testing Setup**: Uses `@testing-library/react`, `@testing-library/jest-dom`, and `@testing-library/user-event` for component testing
- **CRA Configuration**: Standard Create React App setup with react-scripts 5.0.1

## Testing Guidelines

- Test files use the `.test.js` extension
- Tests are colocated with source files in the `src/` directory
- Use React Testing Library's user-centric query methods (`getByRole`, `getByText`, etc.)
- `setupTests.js` imports `@testing-library/jest-dom` for custom matchers

## Important Notes

- This project uses Create React App, so webpack/babel configurations are abstracted
- To modify build configuration, you would need to run `npm run eject` (irreversible)
- The project includes web vitals performance monitoring via `reportWebVitals.js`

## Deployment Pipeline

### GitHub Actions CI/CD
The project includes an automated production deployment pipeline at `.github/workflows/production-deploy.yml`:

**Pipeline stages:**
1. **Test**: Runs all tests with coverage reporting
2. **Build**: Creates optimized production build and artifacts
3. **Deploy**: Uses Ansible to deploy to production servers
4. **Verify**: Performs health checks post-deployment

**Trigger**: Automatic on push to `main` branch, or manual via GitHub Actions UI

### Ansible Deployment
Deployment is handled by Ansible playbooks in the `ansible/` directory:

```bash
# Manual deployment
cd ansible
ansible-playbook -i inventory/production.yml deploy.yml \
  --private-key ~/.ssh/deploy_key \
  --extra-vars "build_artifact=../build.tar.gz"
```

**Ansible roles:**
- `nginx`: Installs and configures nginx web server with optimized settings
- `deploy`: Handles application deployment with release management (keeps last 5 releases)

**Key features:**
- Zero-downtime deployments via symlink switching
- Automatic rollback capability
- Release history management
- Health check verification

### Required GitHub Secrets
- `PRODUCTION_SERVER`: Server IP/hostname (required)
- `PRODUCTION_DOMAIN`: Application domain name (optional - uses IP if not set)
- `SSH_PRIVATE_KEY`: SSH key for server authentication (required)
- `SSH_PORT`: Custom SSH port (optional - defaults to 22)

**Homelab support**: The pipeline works with IP-only setups and custom SSH ports. See `DEPLOYMENT.md` section 4 for homelab configuration example.

**Documentation**: See `DEPLOYMENT.md` for complete setup and usage guide, and `ansible/README.md` for Ansible-specific documentation.
