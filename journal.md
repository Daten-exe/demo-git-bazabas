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

## Étape 5 — README poussé

```
$ git push
To https://github.com/Daten-exe/demo-git-bazabas.git
   2f1caff..1c4f540  main -> main
```

Le commit `1c4f540 Ajoute README.md et journal du TP 05` est visible sur GitHub. `2f1caff..1c4f540`
indique que la branche `main` de GitHub a avancé de l'ancien commit au nouveau.

## Étape 6 — Modification en ligne

Commit fait sur GitHub : `218e488 Add project description to README`.
Juste après, `git status` sur mon poste affiche encore `Your branch is up to date with 'origin/main'`.
C'est normal : mon poste n'a pas recontacté GitHub, donc `origin/main` est encore l'ancienne photo
(`1c4f540`). `git status` compare avec cette photo, pas avec GitHub en direct.

## Étape 7 — Prédiction puis `git pull`

**Prédiction (2026-11-09 16:17) :** *avance rapide (fast-forward)*. Je n'ai fait aucun commit
local depuis mon dernier push : ma `main` (`1c4f540`) est un ancêtre direct de la `main` de GitHub
(`218e488`). Git n'a qu'à avancer le pointeur, sans commit de fusion. Il n'y aura pas de conflit
non plus : ma seule modification locale non commitée concerne `journal.md`, alors que le commit en
ligne ne touche que `README.md`.

**Résultat réel :**

```
$ git pull
From https://github.com/Daten-exe/demo-git-bazabas
   1c4f540..218e488  main       -> origin/main
Updating 1c4f540..218e488
Fast-forward
 README.md | 1 +
 1 file changed, 1 insertion(+)
```

Prédiction confirmée : avance rapide. La sortie montre les deux temps du `pull`. D'abord le `fetch`
met à jour la photo (`1c4f540..218e488  main -> origin/main`), puis la fusion avance ma `main` (`Fast-forward`).

## Étape 8 — `git fetch` seul

Seconde modification en ligne : `0fa2aa1 Add note about second modification`.

```
$ git fetch
$ git status
On branch main
Your branch is behind 'origin/main' by 1 commit, and can be fast-forwarded.

$ git log --oneline --all --graph
* 0fa2aa1 (origin/main, origin/HEAD) Add note about second modification
* 218e488 (HEAD -> main) Add project description to README
* 1c4f540 Ajoute README.md et journal du TP 05
```

Après `git fetch`, le nouveau commit `0fa2aa1` est **téléchargé dans mon dépôt local, mais seulement
sous `origin/main`**. Ma branche `main`, et donc `HEAD`, est restée sur `218e488`. Le `README.md` sur
mon disque n'a pas encore la nouvelle ligne. `git status` le dit : `behind 'origin/main' by 1 commit`.

Différence avec l'étape 7 : `git pull` = `git fetch` **+** `git merge origin/main`. À l'étape 7, il
avait donc aussi avancé `main` et mis à jour mes fichiers. `git fetch` seul ne touche jamais à mes
branches ni à mes fichiers : il met seulement à jour la photo de GitHub. Pour rattraper le retard :
`git merge origin/main` (ou `git pull`).
