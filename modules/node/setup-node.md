# Node.js Setup Guide

This guide walks through the steps to set up Node.js using NVM:
1. Installs curl for downloading resources
2. Installs NVM (Node Version Manager)
3. Installs the latest LTS version of Node.js
4. Installs global dependencies like PM2 and TypeScript
5. Verifies all installations are working correctly

## Install curl
```bash
sudo apt-get install -y curl
```

## Install NVM (Node Version Manager)
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
```

## Load NVM environment
Close and reopen the terminal to load the nvm environment variables.

## Install Node.js LTS version
```bash
nvm install "lts/*"
nvm use "lts/*"
nvm alias default "lts/*"
```

## Install global dependencies
```bash
npm install -g pm2 typescript
```

## Verify installations
```bash
nvm --version
node --version
npm --version
pm2 --version
tsc --version
```
