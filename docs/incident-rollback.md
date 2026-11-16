# Incident de mise en service — retour arrière v0.9.0 → v1.0.0

1. La version défectueuse v0.9.0 a été mise en service sur staging : le conteneur `api` était démarré, mais `GET /health` répondait HTTP 500.
2. Nous l'avons su environ 60 secondes plus tard, lorsque `scripts/wait-healthy.sh` a épuisé ses 12 tentatives et s'est terminé avec `staging non sain` (code 1).
3. Nous avons remis `IMAGE_TAG=v1.0.0` dans `.env.deploy`, relancé `docker compose --env-file .env.deploy -f compose.staging.yaml up -d`, sans reconstruire l'image.
4. Le service a été indisponible environ 70 secondes, entre le démarrage de v0.9.0 et le retour du contrôle de santé positif sur v1.0.0.
5. Pour éviter que cela se reproduise : ne promouvoir qu'un tag dont les tests sont verts, conserver l'image précédente prête, et faire du contrôle de santé un préalable obligatoire avant de valider une mise en service.

## Preuves

Avant l'incident, sur v1.0.0 :

```text
{"status":"ok"}
staging sain
exit=0
```

Pendant l'incident, sur v0.9.0 :

```text
curl: (22) The requested URL returned error: 500
...
staging non sain
exit=1
```

Première erreur utile dans les journaux :

```text
api-1  | 172.18.0.1 - - [16/Nov/2026 10:38:44] "GET /health HTTP/1.1" 500 -
```

Après le retour arrière, sur v1.0.0 :

```text
{"status":"ok"}
staging sain
exit=0
```
