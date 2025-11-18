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
