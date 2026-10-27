# ai-service

IA locale gratuite en Docker (Ollama), exposée par HTTP pour être consommée par vos autres projets.

## Démarrage

```bash
docker compose up -d
```

Au premier lancement, le service `init` télécharge le modèle défini dans `.env` (≈ 2 Go pour `llama3.2:3b`). Les modèles sont persistés dans le volume `ai-service-data`.

Suivi :

```bash
docker compose ps        # init doit passer à "exited (0)" une fois le modèle téléchargé
docker compose logs -f   # voir la progression du téléchargement
```

## Changer de modèle

1. Éditer `.env` :

   ```bash
   MODEL=qwen2.5:7b
   ```

2. Télécharger le nouveau modèle puis nettoyer l'ancien si besoin :

   ```bash
   docker compose run --rm init
   ```

   Liste des modèles disponibles : <https://ollama.com/library>

## API

Base URL sur l'hôte : `http://localhost:11434`

```bash
# Lister les modèles installés
curl http://localhost:11434/api/tags

# Génération
curl http://localhost:11434/api/generate -d '{
  "model": "llama3.2:3b",
  "prompt": "Bonjour !",
  "stream": false
}'

# Chat (compatible OpenAI sur /v1)
curl http://localhost:11434/v1/chat/completions -H 'Content-Type: application/json' -d '{
  "model": "llama3.2:3b",
  "messages": [{"role": "user", "content": "Bonjour !"}]
}'
```

## Utilisation par d'autres projets

### Depuis l'hôte (autres apps, scripts…)

```bash
OLLAMA_HOST=http://localhost:11434
```

### Depuis un autre projet Docker Compose

Le réseau externe `ai-service-net` est créé par ce compose. Ajouter dans le `docker-compose.yml` du projet tiers :

```yaml
services:
  mon-app:
    # ...
    environment:
      OLLAMA_HOST: http://ollama:11434
    networks:
      - ai-service

networks:
  ai-service:
    external: true
    name: ai-service-net
```

Puis, dans le code du projet tiers, appeler `http://ollama:11434` (ex. SDK OpenAI avec `base_url=http://ollama:11434/v1`).

## Commandes utiles

```bash
docker compose up -d        # démarrer
docker compose stop         # arrêter (les modèles restent sur le volume)
docker compose down         # arrêter et supprimer les conteneurs
docker compose exec ollama ollama list        # modèles installés
docker compose exec ollama ollama ps          # modèles chargés en mémoire
```
