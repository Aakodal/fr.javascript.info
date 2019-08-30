```js demo
function throttle(func, ms) {

  let isThrottled = false,
    savedArgs,
    savedThis;

  function wrapper() {

    if (isThrottled) { // (2)
      savedArgs = arguments;
      savedThis = this;
      return;
    }

    func.apply(this, arguments); // (1)

    isThrottled = true;

    setTimeout(function() {
      isThrottled = false; // (3)
      if (savedArgs) {
        wrapper.apply(savedThis, savedArgs);
        savedArgs = savedThis = null;
      }
    }, ms);
  }

  return wrapper;
}
```

Un appel à `throttle(func, ms)` retourne `wrapper`.

1. Lors du premier appel, le `wrapper` exécute simplement `func` et définit l'état de refroidissement (`isThrottled = true`).
2. Dans cet état, tous les appels sont mémorisés dans `savedArgs/savedThis`. Veuillez noter que le contexte et les arguments sont d'égale importance et doivent être mémorisés. Nous avons besoin d'eux simultanément pour reproduire l'appel.
3. ...Ensuite, après le passage de `ms` millisecondes,`setTimeout` se déclenche. L'état de refroidissement est supprimé (`isThrottled = false`). Et si nous avions ignoré les appels, alors `wrapper` est exécuté avec les derniers arguments et contextes mémorisés.

La 3ème étape n’exécute pas `func`, mais `wrapper`, car nous devons non seulement exécuter `func`, mais encore une fois entrer dans l’état de refroidissement et configurer le délai d’expiration pour le réinitialiser.
