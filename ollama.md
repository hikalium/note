```
printf "gemma4:e2b gemma4:e4b gemma4:26b gemma4:31b " | xargs -I {} -d ' ' -- ollama pull {}
```
