# Trace de débogage — Mbaye Sylla

Session effectuée le 3 novembre 2026 (UTC), sur la compilation Debug de Diayma ciblant netcoreapp2.0.

Outil : NetCoreDbg 3.2.0 (archive officielle 3.2.0-1092), piloté par le protocole Debug Adapter Protocol. SDK : 2.1.202 ; runtime de l’application : .NET Core 2.0.9. Cette session n’utilise pas l’interface Visual Studio.

Les arrêts ci-dessous proviennent des événements stopped du débogueur. Les lignes et les méthodes sont celles des piles stackTrace. Les valeurs proviennent des requêtes scopes et variables.

## Cinq points d’arrêt demandés

- Components/CartSummaryViewComponent.cs:12 : atteint.
- Controllers/CartController.cs:15 : atteint.
- Controllers/OrderController.cs:17 : atteint.
- Controllers/ProductController.cs:15 : atteint.
- Startup.cs:20 : atteint.

## Ordre des arrêts et commandes exécutées

Les trois requêtes sont envoyées successivement : GET /, GET /Cart, GET /Order. Les arrêts de CartController et OrderController viennent donc après le premier accueil. Les constructeurs de dépôts réapparaissent parce que leurs services sont enregistrés en transient.

~~~text
01. P2FixAnAppDotNetCode.Program.Main() — P2FixAnAppDotNetCode/Program.cs:10 -> continue
02. P2FixAnAppDotNetCode.Startup..ctor() — P2FixAnAppDotNetCode/Startup.cs:20 -> continue
03. P2FixAnAppDotNetCode.Startup.ConfigureServices() — P2FixAnAppDotNetCode/Startup.cs:28 -> continue
04. P2FixAnAppDotNetCode.Startup.Configure() — P2FixAnAppDotNetCode/Startup.cs:65 -> continue
05. P2FixAnAppDotNetCode.Models.Repositories.ProductRepository..ctor() — P2FixAnAppDotNetCode/Models/Repositories/ProductRepository.cs:15 -> continue
06. P2FixAnAppDotNetCode.Models.Repositories.ProductRepository.GenerateProductData() — P2FixAnAppDotNetCode/Models/Repositories/ProductRepository.cs:24 -> continue
07. P2FixAnAppDotNetCode.Models.Repositories.OrderRepository..ctor() — P2FixAnAppDotNetCode/Models/Repositories/OrderRepository.cs:14 -> continue
08. P2FixAnAppDotNetCode.Models.Services.ProductService..ctor() — P2FixAnAppDotNetCode/Models/Services/ProductService.cs:17 -> continue
09. P2FixAnAppDotNetCode.Controllers.ProductController..ctor() — P2FixAnAppDotNetCode/Controllers/ProductController.cs:15 -> continue
10. P2FixAnAppDotNetCode.Controllers.ProductController.Index() — P2FixAnAppDotNetCode/Controllers/ProductController.cs:21 -> stepIn
11. P2FixAnAppDotNetCode.Models.Services.ProductService.GetAllProducts() — P2FixAnAppDotNetCode/Models/Services/ProductService.cs:25 -> next
12. P2FixAnAppDotNetCode.Models.Services.ProductService.GetAllProducts() — P2FixAnAppDotNetCode/Models/Services/ProductService.cs:29 -> stepIn
13. P2FixAnAppDotNetCode.Models.Repositories.ProductRepository.GetAllProducts() — P2FixAnAppDotNetCode/Models/Repositories/ProductRepository.cs:42 -> next
14. P2FixAnAppDotNetCode.Models.Repositories.ProductRepository.GetAllProducts() — P2FixAnAppDotNetCode/Models/Repositories/ProductRepository.cs:43 -> next
15. P2FixAnAppDotNetCode.Models.Repositories.ProductRepository.GetAllProducts() — P2FixAnAppDotNetCode/Models/Repositories/ProductRepository.cs:44 -> stepOut
16. P2FixAnAppDotNetCode.Models.Services.ProductService.GetAllProducts() — P2FixAnAppDotNetCode/Models/Services/ProductService.cs:29 -> stepOut
17. P2FixAnAppDotNetCode.Controllers.ProductController.Index() — P2FixAnAppDotNetCode/Controllers/ProductController.cs:21 -> next
18. P2FixAnAppDotNetCode.Controllers.ProductController.Index() — P2FixAnAppDotNetCode/Controllers/ProductController.cs:22 -> next
19. P2FixAnAppDotNetCode.Controllers.ProductController.Index() — P2FixAnAppDotNetCode/Controllers/ProductController.cs:23 -> next
20. P2FixAnAppDotNetCode.Components.CartSummaryViewComponent..ctor() — P2FixAnAppDotNetCode/Components/CartSummaryViewComponent.cs:12 -> continue
21. P2FixAnAppDotNetCode.Models.Repositories.ProductRepository..ctor() — P2FixAnAppDotNetCode/Models/Repositories/ProductRepository.cs:15 -> continue
22. P2FixAnAppDotNetCode.Models.Repositories.ProductRepository.GenerateProductData() — P2FixAnAppDotNetCode/Models/Repositories/ProductRepository.cs:24 -> continue
23. P2FixAnAppDotNetCode.Models.Repositories.OrderRepository..ctor() — P2FixAnAppDotNetCode/Models/Repositories/OrderRepository.cs:14 -> continue
24. P2FixAnAppDotNetCode.Models.Services.ProductService..ctor() — P2FixAnAppDotNetCode/Models/Services/ProductService.cs:17 -> continue
25. P2FixAnAppDotNetCode.Controllers.CartController..ctor() — P2FixAnAppDotNetCode/Controllers/CartController.cs:15 -> continue
26. P2FixAnAppDotNetCode.Components.CartSummaryViewComponent..ctor() — P2FixAnAppDotNetCode/Components/CartSummaryViewComponent.cs:12 -> continue
27. P2FixAnAppDotNetCode.Models.Repositories.OrderRepository..ctor() — P2FixAnAppDotNetCode/Models/Repositories/OrderRepository.cs:14 -> continue
28. P2FixAnAppDotNetCode.Models.Repositories.ProductRepository..ctor() — P2FixAnAppDotNetCode/Models/Repositories/ProductRepository.cs:15 -> continue
29. P2FixAnAppDotNetCode.Models.Repositories.ProductRepository.GenerateProductData() — P2FixAnAppDotNetCode/Models/Repositories/ProductRepository.cs:24 -> continue
30. P2FixAnAppDotNetCode.Models.Repositories.OrderRepository..ctor() — P2FixAnAppDotNetCode/Models/Repositories/OrderRepository.cs:14 -> continue
31. P2FixAnAppDotNetCode.Models.Services.ProductService..ctor() — P2FixAnAppDotNetCode/Models/Services/ProductService.cs:17 -> continue
32. P2FixAnAppDotNetCode.Controllers.OrderController..ctor() — P2FixAnAppDotNetCode/Controllers/OrderController.cs:17 -> continue
33. P2FixAnAppDotNetCode.Components.CartSummaryViewComponent..ctor() — P2FixAnAppDotNetCode/Components/CartSummaryViewComponent.cs:12 -> continue
~~~

Équivalences : stepIn = pas à pas détaillé (F11) ; next = pas à pas principal (F10) ; stepOut = pas à pas sortant (Maj+F11) ; continue = poursuivre (F5). Ce sont les commandes effectivement envoyées au débogueur, pas un relevé de touches pressées dans Visual Studio.

## Valeurs observées

- ProductController.Index, ligne 21, avant l’appel : products = null, de type `System.Collections.Generic.List<P2FixAnAppDotNetCode.Models.Product>`.
- ProductController.Index, ligne 21, immédiatement après le retour au point de séquence de l’appel : products = null.
- ProductController.Index, ligne 22, après l’affectation : products contient une instance `System.Collections.Generic.List<P2FixAnAppDotNetCode.Models.Product>`.

Au constructeur de ProductController, les arguments productService et languageService contiennent respectivement les instances ProductService et LanguageService. À CartSummaryViewComponent:12, l’argument cart contient une instance Cart.

## Résultats des requêtes

- GET / : HTTP 200, neuf produits dans le HTML.
- GET /Cart : HTTP 200.
- GET /Order : HTTP 200.

Contrôles automatiques de la session réussis : les cinq points demandés ont été atteints ; stepIn, next et stepOut ont tous été exécutés ; le dépôt a été visité ; les trois requêtes ont abouti et l’accueil contient neuf produits.

La session a ensuite été fermée avec terminaison du processus débogué. Les deux bugs du panier restent ceux documentés dans le README ; aucun correctif métier n’est ajouté par cette session.
