---
nav_title: Notification push sur un domaine alternatif pour le Web
article_title: Notification push sur un domaine alternatif pour le Web
platform: Web
page_order: 20
page_type: reference
description: "Cet article explique comment intégrer les notifications push Braze pour le Web sur un domaine alternatif."
channel: notification push

---

# Notification push sur un domaine alternatif pour le Web

Pour intégrer une notification push Web, votre domaine doit être [sécurisé][2], ce qui signifie généralement `https`, `localhost`, et d’autres exceptions telles que celles définies dans les [normes de notification push W3C][1]. Vous devrez également pouvoir enregistrer un service de traitement à la racine de votre domaine, ou au moins pouvoir contrôler les en-têtes HTTP pour ce fichier.

_Si vous ne pouvez pas répondre à tous ces critères_, utilisez ce guide pour configurer une solution de contournement vous permettant d’ajouter une boîte de dialogue de demande de notification push à votre site Internet. Par exemple, cet article serait utile si vous souhaitez que l’utilisateur s’abonne depuis un site Internet `http` (non sécurisé) ou à partir de la fenêtre contextuelle d’extension du navigateur qui empêche la demande de notification push de s’afficher.

## Mises en garde
Gardez à l’esprit que, comme de nombreuses solutions de contournement sur le Web, les navigateurs évoluent continuellement et, à l’avenir, elle peut ne plus fonctionner comme prévu.

- Cela exige que :
  - Vous possédez un domaine sécurisé séparé (`https://`) et avez accès à un service de traitement sur ce domaine.
  - Les utilisateurs doivent être connectés à votre site Internet pour s’assurer que les jetons de notification push sont liés aux mêmes profils.

{% alert note %}
Vous ne pouvez pas implémenter ainsi de notification push pour Shopify. Shopify prend des mesures pour supprimer les en-têtes qui sont nécessaires pour les notifications push.
{% endalert %}

## Intégration

Pour expliquer l’exemple suivant, nous utiliserons `http://insecure.com` et `https://secure.com` comme nos deux domaines dans le but d’amener les visiteurs à s’inscrire pour les notifications push sur `http://insecure.com`. Cet exemple peut également être appliqué à un système `chrome-extension://` pour la page contextuelle d’une extension de navigateur.

### Étape 1 : Initier le flux de demande

Sur `insecure.com`, ouvrez une nouvelle fenêtre vers votre domaine sécurisé en utilisant un paramètre URL pour transmettre l’ID externe Braze de l’utilisateur actuellement connecté.

**http://insecure.com**
```html
<button id="opt-in">S’abonner aux notifications push</button>
<script>
// the same ID you would use with `braze.changeUser`:
const user_id = getUserIdSomehow();
// pass the user ID into the secure domain URL:
const secure_url = `https://secure.com/push-registration.html?external_id=${user_id}`;

// when the user takes some action, open the secure URL in a new window
document.getElementById("opt-in").onclick = function(){
    if (!window.open(secure_url, 'Opt-In to Push', 'height=500,width=600,left=150,top=150')) {
        window.alert('The popup was blocked by your browser');
    } else {
        // user is shown a popup window
        // and you can now prompt for push in this window
    }
}
</script>
```

### Étape 2 : Enregistrer la notification push

À ce stade, `secure.com` ouvre une fenêtre contextuelle dans laquelle vous pouvez initialiser le SDK Braze pour le Web pour le même ID utilisateur et demander l’autorisation de l’utilisateur pour la notification push Web.

**https://secure.com/push-registration.html**

<script src="https://braze-inc.github.io/embed-like-gist/embed.js?target=https%3A%2F%2Fgithub.com%2Fbraze-inc%2Fbraze-web-sdk%2Fblob%2Fmaster%2Fsnippets%2Falternate-push-domain-registration.html&style=github&showBorder=on&showLineNumbers=on&showFileMeta=on&showCopy=on"></script>

### Étape 3 : Communiquer entre les domaines (facultatif)

Maintenant que les utilisateurs peuvent s’abonner à partir de ce flux de travail provenant de `insecure.com`, vous pouvez modifier votre site selon le fait que l’utilisateur est déjà abonné ou pas. Demander à l’utilisateur de s’enregistrer pour les notifications push alors qu’il l’est déjà est inutile.

 Vous pouvez utiliser iFrames et l’API [`postMessage`][3] pour communiquer entre vos deux domaines. 

**insecure.com**

Sur notre domaine `insecure.com`, nous demanderons au domaine sécurisé (où la notification push est  _vraiment_  enregistrée) des informations sur l’enregistrement de l’utilisateur actuel aux notifications push :

```html
<!-- Create an iframe to the secure domain and run getPushStatus onload-->
<iframe id="push-status" src="https://secure.com/push-status.html" onload="getPushStatus()" style="display:none;"></iframe>

<script>
function getPushStatus(event){
    // send a message to the iframe asking for push status
    event.target.contentWindow.postMessage({type: 'get_push_status'}, 'https://secure.com');
    // listen for a response from the iframe's domain
    window.addEventListener("message", (event) => {
        if (event.origin === "http://insecure.com" && event.data.type === 'set_push_status') {
            // update the page based on the push permission we're told
            window.alert(`Is user registered for push? ${event.data.isPushPermissionGranted}`);
        }
    }   
}
</script>
```

**secure.com/push-status.html**

<script src="https://braze-inc.github.io/embed-like-gist/embed.js?target=https%3A%2F%2Fgithub.com%2Fbraze-inc%2Fbraze-web-sdk%2Fblob%2Fmaster%2Fsnippets%2Falternate-push-domain-status.html&style=github&showBorder=on&showLineNumbers=on&showFileMeta=on&showCopy=on"></script>

[1]: https://www.w3.org/TR/service-workers/#security-considerations
[2]: https://w3c.github.io/webappsec-secure-contexts/
[3]: https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage
