# Ajouter un formulaire de contact à un site Hugo (sans backend)

Exemple complet et clonable : un shortcode Hugo réutilisable pour recevoir les messages de votre formulaire de contact par email, avec [AirMess](https://airmess.fr/?utm_source=github&utm_medium=readme&utm_campaign=airmess-hugo-example&utm_content=intro). Compatible GitHub Pages, Netlify et Cloudflare Pages.

Autres exemples : [HTML](https://github.com/MaximeBranger/airmess-html-example) · [Astro](https://github.com/MaximeBranger/airmess-astro-example)

📖 Tutoriel complet : [https://airmess.fr/tutoriels/formulaire-contact-hugo](https://airmess.fr/tutoriels/formulaire-contact-hugo?utm_source=github&utm_medium=readme&utm_campaign=airmess-hugo-example&utm_content=tutoriel)

## Démarrage rapide

```bash
git clone https://github.com/MaximeBranger/airmess-hugo-example.git
cd airmess-hugo-example
```

1. Remplacez `VOTRE_ID` dans [hugo.toml](hugo.toml) par l'ID de votre formulaire AirMess.
2. Lancez `hugo server` et ouvrez `/contact/`.
3. Ajoutez l'origine locale (ex. `http://localhost:1313`) à la liste des origines autorisées du formulaire.

## Pourquoi c'est compliqué avec Hugo

Hugo génère des fichiers statiques. Il n'y a rien côté serveur pour traiter une soumission. On va donc créer un shortcode qui envoie les données à AirMess, et vous pourrez l'insérer dans n'importe quelle page.

## Étape 1 : déclarer l'URL dans la configuration

Dans `hugo.toml` :

```toml
[params.airmess]
  formUrl = "https://airmess.fr/f/VOTRE_ID"
```

Garder l'URL dans la configuration permet de la changer sans toucher aux templates.

## Étape 2 : créer le shortcode

Créez `layouts/shortcodes/contact.html` :

```html
<form id="contact-form" action="{{ site.Params.airmess.formUrl }}" method="POST">
  <label for="name">Nom</label>
  <input id="name" name="name" type="text" required>
  <label for="email">Email</label>
  <input id="email" name="email" type="email" required>
  <label for="message">Message</label>
  <textarea id="message" name="message" rows="5" required></textarea>
  <button type="submit">Envoyer</button>
  <p id="contact-status" role="status"></p>
</form>

<script>
  const form = document.querySelector('#contact-form');
  const status = document.querySelector('#contact-status');

  form.addEventListener('submit', async (e) => {
    e.preventDefault();
    const button = form.querySelector('button');
    button.disabled = true;
    status.textContent = 'Envoi en cours…';

    try {
      const res = await fetch(form.action, {
        method: 'POST',
        body: new FormData(form), // si l'API attend du JSON : adapter ici
      });
      if (!res.ok) throw new Error(res.status);
      form.reset();
      status.textContent = 'Merci, votre message a bien été envoyé.';
    } catch {
      status.textContent = "L'envoi a échoué. Réessayez dans un instant.";
    } finally {
      button.disabled = false;
    }
  });
</script>
```

## Étape 3 : l'utiliser dans une page

Dans `content/contact.md` :

```markdown
---
title: "Contact"
---

Une question ? Écrivez-moi.

{{< contact >}}
```

## Remarques

Si votre thème bloque les scripts inline via une Content Security Policy, déplacez le script dans `static/js/contact.js` et chargez-le avec une balise `<script src>`. Pour les tests avec `hugo server`, vérifiez que votre origine locale est autorisée.

Si l'envoi échoue avec une erreur CORS dans la console, votre domaine n'est pas dans la liste des origines autorisées. Pour bloquer les robots, activez hCaptcha dans les réglages du formulaire.

## Licence

[MIT](LICENSE)
