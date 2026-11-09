# Journal — TP 05 : Git distant avec GitHub

Dépôt : https://github.com/Daten-exe/demo-git-bazabas

## Étape 3 — `git remote -v` et `git branch -vv`

```
$ git remote -v
origin  https://github.com/Daten-exe/demo-git-bazabas.git (fetch)
origin  https://github.com/Daten-exe/demo-git-bazabas.git (push)

$ git branch -vv
  feature-a 57d47f2 Ajoute ligne 4 (feature-a)
  feature-b 9d6ad56 Ajoute b.txt (feature-b)
  feature-c 20e2f74 Modifie ligne 2 (feature-c)
* main      2f1caff [origin/main] Merge branch 'feature-c'
```

Seule `main` affiche `[origin/main]` : c'est la seule branche poussée avec `-u`. Les branches
`feature-*` sont restées locales, sans branche amont.

**Que signifie `origin/main` ?**
C'est la branche *amont* (upstream) suivie par ma branche `main`. C'est une référence locale, dite de
suivi distant : la copie que mon poste garde de la branche `main` du remote `origin` (GitHub), telle
qu'il l'a vue au dernier `push`, `fetch` ou `pull`. Ce n'est pas GitHub en direct. Si quelqu'un pousse
sur GitHub, `origin/main` ne bouge pas tant que je n'ai pas recontacté le serveur.
Quand `git branch -vv` n'affiche ni `ahead` ni `behind`, `main` et `origin/main` pointent sur le même commit.

**Pourquoi `-u` ne se met qu'une fois ?**
`-u` (`--set-upstream`) écrit le lien `main` → `origin/main` dans `.git/config` :

```
[branch "main"]
        remote = origin
        merge = refs/heads/main
```

Ce réglage est persistant. Ensuite, `git push`, `git pull` et `git status` savent tout seuls avec quoi
comparer et vers où publier. Le remettre ne servirait à rien.

## Étape 4 — Les deux protocoles

```
$ git remote add ssh-origin git@github.com:Daten-exe/demo-git-bazabas.git
$ git ls-remote ssh-origin
git@github.com: Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

| Remote | URL | Ce qu'il faut pour s'authentifier |
|---|---|---|
| `origin` (HTTPS) | `https://github.com/Daten-exe/demo-git-bazabas.git` | Un identifiant GitHub et un **token** (Personal Access Token ou jeton OAuth). GitHub n'accepte plus de mot de passe depuis 2021. Chez moi, c'est Git Credential Manager (`credential.helper=manager`) qui fournit le jeton, enregistré après une connexion dans le navigateur : je n'ai rien eu à taper. Port 443. |
| `ssh-origin` (SSH) | `git@github.com:Daten-exe/demo-git-bazabas.git` | Une **paire de clés** : la clé privée reste sur le poste (`~/.ssh/id_ed25519`, chargée par l'agent SSH), la clé publique doit être déclarée sur le compte (*Settings → SSH and GPG keys*). Ni identifiant ni token. Port 22. |

Constat : ma clé `id_ed25519` existe bien sur le poste, mais elle n'est pas déclarée sur mon compte
GitHub. GitHub refuse donc la connexion (`Permission denied (publickey)`), et `ssh -T git@github.com`
donne le même refus. C'est pour ça que je travaille avec `origin` en HTTPS.
