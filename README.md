1. Write a workflow triggered on push and PR
Create a file at .github/workflows/ci.yml 
name: CI

on:
  push:
    branches: [main]        
  pull_request:
    branches: [main]        

2. Add build and test jobs

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install dependencies
        run: npm install

      - name: Build
        run: npm run build

  test:
    runs-on: ubuntu-latest
    needs: build             # waits for build to pass first
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm install
      - name: Run tests
        run: npm test

3. Cache dependencies for speed
  
  - name: Set up Node.js with cache
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'         
For Docker builds, add BuildKit layer caching too:
- name: Build Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          cache-from: type=gha      
          cache-to: type=gha,mode=max   

4. Inject secrets securely where needed
use GitHub Secrets
Set them up:
GitHub repo → Settings → Secrets and variables → Actions → New repository secret

Add secrets like:
DB_PASSWORD = yourpassword
API_KEY     = yourapikey

Reference them in the workflow 
- name: Run tests
        run: npm test
        env:
          DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
          API_KEY: ${{ secrets.API_KEY }}

5. Make the check required to merge
GitHub repo → Settings → Branches → Branch protection rules → Edit rule for main

Full workflow file
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'
      - run: npm install
      - run: npm run build

  test:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'
      - run: npm install
      - name: Run tests
        run: npm test
        env:
          DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
          API_KEY: ${{ secrets.API_KEY }}

        <img width="1536" height="1024" alt="ss101" src="https://github.com/user-attachments/assets/14f190dc-9263-4094-b2ad-a4c35c8a408b" />
