### 1. Install hugo
```bash
brew install hugo
```

### 2. Start development server

```bash
hugo serve --buildDrafts
```

### 3. Release

run `hugo` to build static dir `public`, then use nginx to serve `public`. 
