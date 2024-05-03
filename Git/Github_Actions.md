# 📚 GitHub Actions - Integración continua (CI/CD) 🚀

## 1. Inicio

Comenzamos clonando desde terminal en VSC el proyecto. 

``` 
git clone rutadelproyectoengithub
``` 

Instalamos dependencias:

``` 
npm install
``` 

Para ver que scripts tenemos:

``` 
npm run
``` 

Y para ejecutar alguno de ellos:

``` 
npm nombre_script
``` 

O 

``` 
npm run nombre_script
``` 

**IMPORTANTE=** Si vamos a trabajar con archivos en formato .yaml es fundamental descargar la extensión YAML en VSC. 

## 2. Pipeline al completo

Dentro de esta ruta: pokedex-for-ci/.github/workflows
/pipeline.yml

``` 
name: Deployment Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
    types: [opened, synchronize]

jobs:

  avoid_reduncy:
    runs-on: ubuntu-18.04
    steps:
      - name: Cancel Previous Redundant Builds
        uses: styfle/cancel-workflow-action@0.9.1
        with:
          access_token: ${{ github.token }}

  lint:
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v2
        with:
          fetch-depth: 0
      - uses: actions/setup-node@v2
        with:
          cache: 'npm'
          node-version: '14'
      - name: Install dependencies
        run: npm ci
      - name: Lint
        run: npm run eslint

  build:
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v2
        with:
          fetch-depth: 0
      - uses: actions/setup-node@v2
        with:
          cache: 'npm'
          node-version: '14'
      - name: Install dependencies
        run: npm ci
      - name: Build
        run: npm run build
      - uses: actions/upload-artifact@v2
        with:
          name: dist
          path: dist

  test:
    needs: [lint, build]
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v2
        with:
          fetch-depth: 0
      - uses: actions/setup-node@v2
        with:
          cache: 'npm'
          node-version: '14'
      - name: Install dependencies
        run: npm ci
      - uses: actions/download-artifact@v2
        with:
          name: dist
          path: dist
      - name: Test
        run: npm test

  e2e:
    needs: [lint, build]
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v2
        with:
          fetch-depth: 0
      - uses: actions/setup-node@v2
        with:
          cache: 'npm'
          node-version: '14'
      - name: Install dependencies
        run: npm ci
      - uses: actions/download-artifact@v2
        with:
          name: dist
          path: dist
      - name: E2E tests
        uses: cypress-io/github-action@v2
        with:
          command: npm run test:e2e
          start: npm run start-test
          wait-on: http://localhost:5000

  deploy:
    needs: [test, e2e]
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v2
        with:
          fetch-depth: 0
      - name: Deploy to Heroku
        if: ${{ github.event_name == 'push' }}
        uses: akhileshns/heroku-deploy@v3.12.12
        with:
          heroku_api_key: ${{secrets.HEROKU_API_KEY}}
          heroku_app_name: ${{secrets.HEROKU_APP}}
          heroku_email: ${{secrets.HEROKU_API_EMAIL}}
          healthcheck: "https://${{secrets.HEROKU_APP}}.herokuapp.com/health"
          rollbackonhealthcheckfailed: true
``` 





