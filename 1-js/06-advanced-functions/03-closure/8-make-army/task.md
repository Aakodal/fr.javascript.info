importance: 5

---

# Armée de fonctions

Le code suivant crée un tableau de `shooters`.

Chaque fonction est censée sortir son numéro. Mais quelque chose ne va pas …

```js run
function makeArmy() {
  let shooters = [];

  let i = 0;
  while (i < 10) {
    let shooter = function() { // shooter function
      alert( i ); // devrait afficher son numéro
    };
    shooters.push(shooter);
    i++;
  }

  return shooters;
}

let army = makeArmy();

army[0](); // the shooter numéro 0 affiche 10
army[5](); // et le numéro 5 affiche aussi 10...
// ... tous les shooters affichent 10 au lieu de 0, 1, 2, 3 …
```

<<<<<<< HEAD
Pourquoi tous les shooters affichent la même chose ? Corrigez le code pour qu’ils fonctionnent comme prévu.
=======
Why do all of the shooters show the same value? Fix the code so that they work as intended.
>>>>>>> 47d186598add3a0ea759615596a12e277ce8fb5a

