name: CI Build
on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
jobs:
  build:
    runs-on: ubuntu-latest
    container: python:3.9-slim

    services:
      postgres:
        image: postgres:alpine
        ports:
          - 5432:5432
        env:
          POSTGRES_PASSWORD: pgs3cr3t
          POSTGRES_DB: testdb
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - name: Checkout
        uses: actions/checkout@v2
      
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip wheel
          pip install -r requirements.txt
      
      - name: Lint with flake8
        run: |
          flake8 service --count --select=E9,F63,F7,F82 --show-source --statistics
          flake8 service --count --max-complexity=10 --max-line-length=127 --statistics
      
      - name: Run unit tests with nose
        run: nosetests
        env:
          DATABASE_URI: "postgresql://postgres:pgs3cr3t@postgres:5432/testdb"
