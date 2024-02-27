# FirstStart
name: Build and Deploy to GitHub pages
on:
  schedule:
    - cron: '0 9 * * *'
  push:
    paths-ignore:
      - 'dist/index.html'
      - 'README.md'
    branches: [ "master" ]
  pull_request:
    branches: [ "master" ]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v2

    - name: Generate static files
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
    - run: npm ci
    - run: npm run build

    - name: Init new repo in dist folder and commit generated files
      run: |
        cd dist/
        git init
        git add -A
        git config --global --add safe.directory /github/workspace
        git config --local user.email "email@email.com"
        git config --local user.name "username"
        git commit -m 'deploy'
        
    - name: Force push to destination branch
      uses: ad-m/github-push-action@master
      with:
        github_token: ${{ secrets.GITHUB_TOKEN }}
        branch: gh-pages
        force: true
        directory: dist
