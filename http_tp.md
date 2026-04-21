# TP : Le Protocole HTTP - Travaux Pratiques

## TP 1 : Exploration avec les DevTools

### 1.2 Observer une requête simple
- **Quel est le code de statut ?** 
  `200 OK`
- **Quels headers de requête sont envoyés ?** 
  Typiquement : `Host`, `User-Agent`, `Accept`, `Accept-Language`, `Accept-Encoding`, `Connection`, `Sec-Fetch-Dest`, `Sec-Fetch-Mode`, `Sec-Fetch-Site`.
- **Quel est le Content-Type de la réponse ?** 
  `application/json`

### 1.4 Observer les codes de statut
| URL | Méthode | Code | Content-Type |
| :--- | :--- | :--- | :--- |
| httpbin.org/get | GET | 200 | application/json |
| httpbin.org/post | POST | 200 | application/json |
| httpbin.org/status/201 | GET | 201 | text/html; charset=utf-8 |

---

## TP 2 : Maîtrise de cURL

### 2.1 Requête GET simple
- **Quelle est la différence entre -i et -v ?**
  - `-i` (`--include`) : Affiche uniquement les headers HTTP de la **réponse** en plus du corps (body) de la réponse.
  - `-v` (`--verbose`) : Affiche l'ensemble des détails de la transaction (résolution DNS, handshake TLS, headers de la **requête** envoyée précédés de `>`, et headers de la **réponse** reçue précédés de `<`).

### 2.5 Exercice avancé
```bash
curl -X POST https://httpbin.org/post \
  -H "Content-Type: application/json" \
  -H "X-Custom-Header: MonHeader" \
  -d '{"action": "test", "value": 42}' \
  -i
```

**Résultat attendu (Headers et Body) :**
```http
HTTP/1.1 200 OK
Date: Tue, 21 Apr 2026 15:44:08 GMT
Content-Type: application/json
Content-Length: 504
Server: gunicorn/19.9.0
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true

{
  "args": {}, 
  "data": "{\"action\": \"test\", \"value\": 42}", 
  "files": {}, 
  "form": {}, 
  "headers": {
    "Accept": "*/*", 
    "Content-Length": "31", 
    "Content-Type": "application/json", 
    "Host": "httpbin.org", 
    "User-Agent": "curl/8.5.0", 
    "X-Custom-Header": "MonHeader"
  }, 
  "json": {
    "action": "test", 
    "value": 42
  }, 
  "origin": "105.157.240.203", 
  "url": "https://httpbin.org/post"
}
```

---

## TP 3 : API REST avec JavaScript

### Exercice pratique
```javascript
async function fetchWithRetry(url, options = {}, maxRetries = 3) {
  let tentatives = 0;
  
  while (tentatives < maxRetries) {
    try {
      const response = await fetch(url, options);
      
      // En cas d'erreur 5xx (Erreur Serveur), on lève une exception 
      // pour déclencher le catch et réessayer
      if (response.status >= 500 && response.status < 600) {
        throw new Error(`HTTP Error ${response.status}`);
      }
      
      // Si ce n'est pas une erreur 5xx (ex: 200 OK, ou même 404), on retourne la réponse
      return response;
      
    } catch (error) {
      tentatives++;
      
      if (tentatives === maxRetries) {
        // Lève l'erreur finale si le nombre maximum de tentatives est atteint
        throw error;
      }
      
      console.warn(`Tentative ${tentatives} échouée. Nouvel essai dans 1 seconde...`);
      // Attend 1 seconde avant de réessayer
      await new Promise(resolve => setTimeout(resolve, 1000));
    }
  }
}

// Utilisation :
// fetchWithRetry('https://httpbin.org/status/500').then(...).catch(...);
```

---

## TP 4 : Analyse des Headers de Sécurité

### Exercice
| Site | HSTS | X-Frame | CSP | Note (Security Headers) |
| :--- | :--- | :--- | :--- | :--- |
| **github.com** | `max-age=31536000; includeSubdomains; preload` | `deny` | `default-src 'none'; ...` | **A+** |
| **google.com** | `max-age=31536000` | `SAMEORIGIN` | `object-src 'none'; ...` | **A** |
| **stackoverflow.com** | `max-age=31536000` | `SAMEORIGIN` | `upgrade-insecure-requests; ...` | **A** |

---

## TP 5 : Cache HTTP

### Exercice
Création d'une page HTML avec image, CSS et JS :

```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>TP 5 - Cache HTTP</title>
    
    <!-- Fichier CSS -->
    <link rel="stylesheet" href="style.css">
    
    <!-- Fichier JS -->
    <script src="script.js" defer></script>
</head>
<body>
    <h1>Test de Cache HTTP</h1>
    <!-- Une image -->
    <img src="image.png" alt="Test Cache">
</body>
</html>
```

Pour configurer les headers de cache appropriés pour chaque type de fichier (côté serveur, via `.htaccess`, nginx.conf, ou Backend) :

**1. Une image (`image.png`) & Un fichier CSS (`style.css`)**  
Pour des ressources statiques qui évoluent rarement :
- Configuration : `Cache-Control: public, max-age=31536000, immutable`
- *Explication :* On applique un cache très long (1 an) pour que le navigateur ne retélécharge jamais ces fichiers, accélérant drastiquement le chargement.

**2. Un fichier JS (`script.js`)**  
Si le fichier peut être mis à jour régulièrement :
- Configuration : `Cache-Control: no-cache` ou `Cache-Control: public, max-age=3600, must-revalidate`
- *Explication :* `no-cache` oblige le navigateur à valider avec le serveur (via `ETag` ou `Last-Modified`) si le fichier a changé. Si c'est le même, le serveur répond `304 Not Modified`, sinon il envoie le nouveau fichier. Cela évite d'exécuter du vieux code JavaScript si une nouvelle version est déployée.
