# Tuto email Symfony

Pour commencer, il vous faut un mail catcher pour éviter d'envoyer un message à une adresse existante sans faire exprès. Je vous laisse le choix entre `Maildev` et `Mailhog`.

## Maildev

Pour Maildev, il vous faut node.js d'installer sur votre ordinateur.

- vérifier que node est installé: `node -v`
- pour installer node.js sur un environnement linux: `sudo apt update` puis `sudo apt install nodejs`
- pour windows vous pouvez télécharger l'exécutable sur le site [nodejs.org](https://nodejs.org/en/download/)
-	installation de maildev: `sudo npm install -g maildev`
-	démarrer Maildev (dans le terminal, n'importe où): `maildev`

## Mailhog

-	télécharger le fichier qui correspond à votre système d'exploitation (pour la VM -> "MailHog_linux_am64") : [github mailhog](https://github.com/mailhog/MailHog/releases/tag/v1.0.1)
-	pour démarrer MailHog : aller à l'endroit où le fichier a été téléchargé et faite : `./MailHog_linux_amd64` (`./`  correspond au dossier courant et `MailHog_linux_amd64`  au nom du fichier téléchargé, donc à ajuster en fonction du votre)

Si vous rencontrez un problème de permission, faite : `chmod 777 MailHog_linux_amd6` (c'est barbare mais ça devrait régler le problème) , puis réexécuter la commande `./MailHog_linux_amd6`

## Interface graphique du mail catcher

Pour accéder à `Maildev` à partir de votre navigateur : `localhost:1080`.

Pour accéder à `Mailhog` à partir de votre navigateur : `localhost:8025`.

## Mailer Symfony

Maintenant il vous faut `mailer` pour envoyer une mail avec symfony.

-	vérifier qu'i n'est pas déjà présent dans vos dépendances avec un CTRL + F de `symfony/mailer` dans votre `composer.json`
-	s'il n'est pas installé: `composer required symfony/mailer`

### Paramétrage de mailer

•	config -> packages -> mailer.yaml, vérifier qu'il y a :
```yaml
// Dans config/packages/mailer.yaml

framework:
    mailer:
        dsn: '%env(MAILER_DSN)%'
```
•	dans le `.env`:`MAILER_DSN=XXXX`
dans le `.env.local`: `MAILER_DSN=smtp://localhost:1025`
Logiquement maildev et mailhog sont sur le port 1025, vous le voyez lors du démarrage. Si ce n'est pas le cas, penser à modifier le port de `MAILER_DNS`.

Pour éviter que Symfony n'envoi tous vos mail dans la BDD, rendez-vous dans le fichier `config\packages\messenger.yaml`, puis commenter la ligne 29 `Symfony\Component\Mailer\Messenger\SendEmailMessage: async`

Je ne sais pas si c'est une bonne pratique de faire cela, mais pour la partir dev ce n'est pas gênant.

### Création d'un mail avec symfony

Il va falloir créer une instance de `Email()`  ou une instance de `TemplatedEmail()` (si vous voulez utiliser une vue twig pour la mise en page de votre mail)

#### Email()
```php
        $email = (new Email())
            ->from('hello@example.com')
            ->to('you@example.com')
            //->cc('cc@example.com')
            //->bcc('bcc@example.com')
            //->replyTo('fabien@example.com')
            //->priority(Email::PRIORITY_HIGH)
            ->subject('Time for Symfony Mailer!')
            ->text('Sending emails is fun again!')
            ->html('<p>See Twig integration for better HTML integration!</p>');
```
#### TemplatedEmail()

```php
        $email = (new TemplatedEmail())
            ->from('hello@example.com')
            ->to('you@example.com')
            //->cc('cc@example.com')
            //->bcc('bcc@example.com')
            //->replyTo('fabien@example.com')
            //->priority(Email::PRIORITY_HIGH)
            ->subject('Time for Symfony Mailer!')
            ->htmlTemplate('mails/mail_contact.html.twig')
            ->context([
                'nomVariableTwig' => $variableAEnvoyerATwig,
            ]);
```

P.S : `(new Email())` est un raccourci pour ne pas avoir à réécrire `$email->from('...')`. Vous pouvez chaîner les méthodes directement sur l'instanciation de la classe à partir du moment où elle retournent `$this`.

### Envoi du mail

Maintenant que le mail est préparé, il faut l'envoyer grâce à `MailerInterface` que vous pouvez injecter directement dans la méthode si vous êtes dans un controller ou passer par le `__construct` si vous êtes dans un service.

On va utiliser la méthode `send()`  de `MailerInterface` : `$mailerInterface->send($email)`;
En cas d'erreur, il renvoie un objet de type `TransportExceptionInterface` , il peut donc être intéressant de faire un `try catch`:
```php
        try{
            $mailerInterface->send($email);
        } catch(TransportExceptionInterface $error){
            echo $error;
        } 
```
Voilà j'espère que c'est assez clair, si vous avez des questions n'hésitez pas!
