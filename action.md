# Testing Workflow

name: Example

on:
  push:
    branches: [ testing ]


<!-- Print Somthing -->
jobs:
  example:
    runs-on: ubuntu-latest
    steps:
      - name: Print
        run: echo "Hello World"
      - name: Print
        run: echo "::debug::This is a debug message"


    # Print repo details using normal console log
    steps:
      - name: Print repo details
        run: |
          echo "Owner: ${{ github.repository_owner }}"
          echo "Repo: ${{ github.event.repository.name }}"
          echo "Event: ${{ github.event_name }}"
          echo "Branch: ${{ github.ref }}"
          echo "Actor: ${{ github.actor }}"
          echo "SHA: ${{ github.sha }}"
          echo "Token: ${{ secrets.GITHUB_TOKEN }}"


<!-- Build node project -->
jobs:
  example:
    runs-on: ubuntu-latest
    # Build Project
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v6

      - name: Install dependencies
        run: npm ci

      - name: Build project
        run: npm run build
        env: 
          ENCRYPTION_KEY: ${{ secrets.ENCRYPTION_KEY }}
      
      - name: Test build
        run: npm run test
        env: 
          ENCRYPTION_KEY: ${{ secrets.ENCRYPTION_KEY }}


<!-- set node v -->
        with:
            node-version: '22'
            cache: 'npm'

<!--  Add cache -->
      - name: Cache Next.js build
        uses: actions/cache@v6
        with:
          path: .next/cache
          key: ${{ runner.os }}-nextjs-${{ hashFiles('**/package-lock.json') }}-${{ hashFiles('**/*.ts', '**/*.tsx', '**/*.js', '**/*.jsx', '!**/node_modules/**') }}
          restore-keys: |
            ${{ runner.os }}-nextjs-${{ hashFiles('**/package-lock.json') }}-
            ${{ runner.os }}-nextjs-



<!--  Upload build artifacts -->
      - name: Upload build artifact
        uses: actions/upload-artifact@v6
        with:
          name: next-build
          path: .next
          include-hidden-files: true
          retention-days: 1 

<!-- Test -->

  test:
    runs-on: ubuntu-latest
    needs: build
    steps :
      - name: Checkout repository
        uses: actions/checkout@v6

      - name: Set up Node.js
        uses: actions/setup-node@v6
        with:
          node-version: '22'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Download build artifact
        uses: actions/download-artifact@v4
        with:
          name: next-build
          path: .next
      
      - name: Run unit tests
        run: npm run test
        env: 
          ENCRYPTION_KEY: ${{ secrets.ENCRYPTION_KEY }}

<!-- security audit -->


  security-audit:
    runs-on: ubuntu-latest
    needs: test
    steps :
      - name: Checkout repository
        uses: actions/checkout@v6

      - name: Set up Node.js
        uses: actions/setup-node@v6
        with:
          node-version: '22'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run security audit
        run: npm audit