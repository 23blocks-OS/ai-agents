---
name: deploy-cloud
description: Deploy applications to 23blocks shared cloud platform
---

# Deploy to 23blocks Cloud

Deploy your application to the 23blocks managed cloud platform.

## Deployment Methods

### CLI Deployment
```bash
# Install 23blocks CLI
npm install -g @23blocks/cli

# Login
23blocks login

# Deploy
23blocks deploy
```

### Git-based Deployment
```bash
# Connect repository
23blocks git:remote -a your-app-name

# Deploy via git push
git push 23blocks main
```

## Configuration

### 23blocks.json
```json
{
  "name": "my-app",
  "stack": "node-18",
  "build": {
    "command": "npm run build"
  },
  "run": {
    "command": "npm start",
    "port": 3000
  },
  "env": {
    "NODE_ENV": "production"
  }
}
```

### Environment Variables
```bash
# Set env vars
23blocks config:set DATABASE_URL=postgres://...
23blocks config:set API_KEY=your-key

# View config
23blocks config
```

## Supported Stacks

- `node-18`, `node-20`: Node.js applications
- `ruby-3.2`: Ruby/Rails applications
- `python-3.11`: Python applications
- `static`: Static sites

## Scaling

```bash
# Scale web dynos
23blocks ps:scale web=2

# View current scale
23blocks ps
```

## Logs

```bash
# View logs
23blocks logs --tail

# View specific process
23blocks logs --ps web.1
```

## Usage

When deploying to 23blocks cloud:
1. Ensure 23blocks.json is configured
2. Set required environment variables
3. Run deployment command
4. Monitor logs for issues
5. Verify application health
