# Build Requirements

This document outlines the build requirements and process for the React Playground project.

## Prerequisites

- **Node.js**: Version 18.x or higher recommended
- **npm**: Version 9.x or higher (comes with Node.js)

## Dependencies

The project uses the following key dependencies:

### Production Dependencies
- `react`: ^19.1.0
- `react-dom`: ^19.1.0

### Development Dependencies
- `vite`: ^7.0.4 - Build tool and dev server
- `typescript`: ~5.8.3 - TypeScript compiler
- `@vitejs/plugin-react-swc`: ^3.10.2 - React plugin for Vite with SWC
- `eslint`: ^9.30.1 - Linting tool
- `typescript-eslint`: ^8.35.1 - TypeScript ESLint rules

## Build Process

### 1. Install Dependencies

```bash
npm install
```

This installs all required dependencies listed in `package.json`.

### 2. Linting

```bash
npm run lint
```

Runs ESLint to check code quality and enforce coding standards.

### 3. Build for Production

```bash
npm run build
```

This command performs two steps:
1. TypeScript compilation (`tsc -b`)
2. Vite production build (`vite build`)

The build output is generated in the `dist/` directory.

### 4. Development Server

```bash
npm run dev
```

Starts the Vite development server with hot module replacement (HMR).

### 5. Preview Production Build

```bash
npm run preview
```

Serves the production build locally for testing.

## Build Output

After running `npm run build`, the following files are generated in the `dist/` directory:

- `index.html` - Main HTML entry point
- `assets/` - Directory containing:
  - JavaScript bundle(s)
  - CSS bundle(s)
  - Static assets (images, fonts, etc.)

## Build Verification

To verify the build was successful:

1. Check that the `dist/` directory exists
2. Verify `dist/index.html` is present
3. Confirm assets are in `dist/assets/`
4. No TypeScript compilation errors

## CI/CD Integration

The build can be integrated into CI/CD pipelines using:

```yaml
- run: npm install
- run: npm run lint
- run: npm run build
```

## Notes

- The `node_modules/` and `dist/` directories are excluded from version control (see `.gitignore`)
- Build artifacts are optimized for production with minification and tree-shaking
- The project uses SWC for React Fast Refresh during development
