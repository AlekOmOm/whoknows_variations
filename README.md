# Whoknows Variations


Use the `ours` merge strategy to merge the `main` branch into the `new_branch` branch. only update the `main` branch.

cd into the src folder where `docker-compose.yml` is located and run the following command:

```bash
git checkout new_branch
git merge -s ours main
```

## How to get started

Each branch is a tutorial in a different topic based on the same Flask application as in the `main` branch. 

One way to follow along is by:

1. Forking the repository to your own account.

2. Cloning the repository to your local machine.

3. Checking out the branch you are interested in (e.g. `git checkout <branch_name>`).

4. Following the instructions in the README of the branch.

5. You can now push changes to your own repository. 

## Further work

If you are interested you can go to the Prometheus dashboard (http://localhost:9090/) and try out the *Prometheus query language*.

Here are some links for inspiration:

https://promlabs.com/promql-cheat-sheet/

https://blog.ruanbekker.com/cheatsheets/prometheus/