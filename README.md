# TP .NET — Mbaye Sylla

**Étudiant : Mbaye Sylla — ESP / UCAD**  
**Année universitaire : 2026/2027 — TP du 3 novembre 2026**  
**Compte GitHub : [sylla-mbaye](https://github.com/sylla-mbaye)**

Je présente mon travail sur Boutique Diayma, à partir du dépôt de l’enseignant [devehod/BoutiqueDiayma2025](https://github.com/devehod/BoutiqueDiayma2025), commit 22e3a29b71e7b17228f053aca8ac2bc5672f82c6.

J’ai personnalisé le titre, la navigation et le pied de page, compilé l’application et exécuté sa publication Windows. J’ai reproduit les deux bugs ci-dessous par HTTP, avec deux clients indépendants. Le code métier initial est conservé.

> **Vérification du débogage :** les cinq points d’arrêt ont été posés et atteints pendant une session réelle NetCoreDbg, avec pas à pas détaillé, principal et sortant. Les piles, les valeurs et l’ordre observé sont consignés dans [TRACE-DEBOGAGE.md](TRACE-DEBOGAGE.md). L’outil utilise le protocole DAP compatible avec les débogueurs de VS Code ; cette session ne repose pas sur l’interface Visual Studio.

## 1. Récupération et exécution

La solution **Diayma.sln** a été récupérée, le projet a été compilé avec le SDK **2.1.202**, puis exécuté en publication Windows autonome. Une seconde publication en configuration **Debug**, avec symboles, a servi à la session de débogage décrite aux questions 7 et 8. Les requêtes vers l’accueil, le panier et la commande ont toutes répondu HTTP 200.

Le débogage a été effectué avec **NetCoreDbg**, via le protocole **DAP**, les outils Visual Studio n’étant pas installés dans cet environnement. Aucun avertissement d’interface Visual Studio n’a donc été observé ni présenté comme fermé. Pour ouvrir la solution dans Visual Studio : charger Diayma.sln avec la charge de travail ASP.NET et web, sélectionner Diayma comme projet de démarrage et lancer en Debug avec F5.

## 2. Projets de la solution

La solution contient un seul projet : **Diayma**, dans [P2FixAnAppDotNetCode/Diayma.csproj](P2FixAnAppDotNetCode/Diayma.csproj). C’est une application web ASP.NET Core MVC. Les dossiers Controllers, Models et Views ne sont pas des projets distincts.

## 3. Framework et SDK

Le projet déclare **Microsoft.NET.Sdk.Web** et cible **netcoreapp2.0**, soit .NET Core 2.0. Ses références incluent Microsoft.AspNetCore.All 2.0.6 et Microsoft.Extensions.Localization 2.1.1.

La version cible ne permet pas de déduire le numéro exact du SDK : le dépôt initial ne contient pas de global.json. Pour ma compilation, le SDK utilisé est **2.1.202**, accompagné du runtime **Microsoft.NETCore.App 2.0.9**. Le numéro du SDK est distinct du framework cible. Les commandes dotnet --version et dotnet --info permettent de relever l’environnement effectif.

## 4. Installation

Le SDK 2.1.202 Windows x64 a été installé par extraction dans le dossier de travail, sans modification de l’installation système. L’empreinte SHA-512 de l’archive a été vérifiée contre les [métadonnées officielles Microsoft](https://dotnetcli.blob.core.windows.net/dotnet/release-metadata/2.0/releases.json). Un runtime seul ne permet pas de compiler. Cette version historique est hors support et réservée ici à l’exercice local.

## 5. Dépôt personnel

Mon dépôt public est [sylla-mbaye/TP-Diayma-Debogage](https://github.com/sylla-mbaye/TP-Diayma-Debogage). Le code initial reste dans l’historique ; mes livrables sont regroupés sur [develop](https://github.com/sylla-mbaye/TP-Diayma-Debogage/tree/develop), par une pull request.

## 6. Deux bugs trouvés et reproduits

### Bug 1 : le total ignore les quantités

J’ai ajouté deux fois le produit 1, XIAOMI Redmi 13C, de prix 88 000. La réponse HTTP de /Cart indique **quantité 2**, **sous-total $176,000.00**, mais **total $88,000.00**. Le total attendu est 176 000. Pour reproduire manuellement : partir d’un panier vide, ajouter deux fois ce produit puis ouvrir le panier.

Cause : [Cart.GetTotalValue(), ligne 66](P2FixAnAppDotNetCode/Models/Cart.cs#L66) additionne uniquement les prix, sans multiplier par Quantity. [AddItem, ligne 40](P2FixAnAppDotNetCode/Models/Cart.cs#L40) incrémente pourtant la quantité. La vue du panier calcule correctement le sous-total avec quantité × prix, puis affiche le total incorrect calculé par cette méthode. Le symbole monétaire dépend de la culture active.

### Bug 2 : les utilisateurs partagent le même panier

Deux clients HTTP avec des CookieJar indépendantes ont été utilisés sur la même instance du serveur. Après les ajouts de A, B retrouve les mêmes articles dans /Cart, alors que son panier devrait être vide. Pour reproduire manuellement : ajouter un article dans une session A et consulter /Cart dans une session privée B.

Cause : [Startup.cs, ligne 29](P2FixAnAppDotNetCode/Startup.cs#L29) enregistre ICart/Cart en **singleton**. Tous les contrôleurs reçoivent donc le même objet contenant la liste mutable. AddSession() ne transforme pas automatiquement cet objet en panier de session ; Cart ne consulte pas la session HTTP.

Les deux bugs sont conservés pour l’exercice. Le panier de test a été vidé après la vérification. Le relevé figure également dans l’archive Windows, fichier VERIFICATION.txt.

## 7. Points d’arrêt

Les cinq points ci-dessous ont été posés et **tous atteints** dans la session réelle. Les valeurs et les piles correspondantes sont dans [la trace](TRACE-DEBOGAGE.md). Pour les rejouer dans Visual Studio, F9 place un point sur ces instructions exécutables :

- [CartSummaryViewComponent.cs:12](P2FixAnAppDotNetCode/Components/CartSummaryViewComponent.cs#L12) : affectation du panier dans le constructeur.
- [ProductController.cs:15](P2FixAnAppDotNetCode/Controllers/ProductController.cs#L15) : affectation du service des produits.
- [OrderController.cs:17](P2FixAnAppDotNetCode/Controllers/OrderController.cs#L17) : affectation du panier.
- [CartController.cs:15](P2FixAnAppDotNetCode/Controllers/CartController.cs#L15) : affectation du panier.
- [Startup.cs:20](P2FixAnAppDotNetCode/Startup.cs#L20) : affectation de Configuration.

Lancer en configuration Debug avec F5 et vérifier que les symboles sont chargés. Startup est atteint au démarrage du processus. CartController et OrderController nécessitent des requêtes vers leurs contrôleurs ; ils ne sont pas systématiquement visités à l’accueil.

## 8. Namespaces, classes, méthodes et pas à pas

Le parcours du code ci-dessous est complété par une **session réelle de débogage**. L’ordre des arrêts, les méthodes et les valeurs observées sont disponibles dans [TRACE-DEBOGAGE.md](TRACE-DEBOGAGE.md). Le pas à pas a suivi ProductController.Index vers ProductService.GetAllProducts puis ProductRepository.GetAllProducts, et le retour vers le contrôleur. Le détail des opérations internes de MVC et Razor reste décrit à partir du source, sans inventer de points d’arrêt dans le framework.

### Démarrage

Dans **P2FixAnAppDotNetCode**, Program.Main(string[] args) appelle BuildWebHost(args).Run(). Program.BuildWebHost utilise WebHost.CreateDefaultBuilder(args), `UseStartup<Startup>()` et Build(). [Source : Program.cs](P2FixAnAppDotNetCode/Program.cs).

Le framework instancie Startup : son constructeur reçoit IConfiguration et affecte Configuration. ConfigureServices(IServiceCollection) enregistre localisation, panier, services, dépôts, cache, session et MVC. Configure(IApplicationBuilder, IHostingEnvironment) installe le pipeline : fichiers statiques, localisation, session puis MVC. La route par défaut est **{controller=Product}/{action=Index}/{id?}**. [Source : Startup.cs](P2FixAnAppDotNetCode/Startup.cs).

AddSingleton et AddTransient précisent les durées de vie : ils ne prouvent pas que les constructeurs sont exécutés immédiatement dans l’ordre des inscriptions.

### Résolution du contrôleur d’accueil

La requête / sélectionne **P2FixAnAppDotNetCode.Controllers.ProductController.Index()**. Le constructeur de ProductController reçoit IProductService et ILanguageService. Le conteneur fournit ProductService et LanguageService, dans **P2FixAnAppDotNetCode.Models.Services**.

ProductService reçoit IProductRepository et IOrderRepository. Leurs implémentations, **ProductRepository** et **OrderRepository**, sont dans **P2FixAnAppDotNetCode.Models.Repositories**. ProductRepository() crée la liste et appelle GenerateProductData(), qui construit les neuf **P2FixAnAppDotNetCode.Models.Product** avec identifiant, stock, prix, nom et description. OrderRepository() initialise sa liste de commandes ; aucune sauvegarde de commande n’est effectuée par Index(). La présence de LanguageService dans les dépendances ne signifie pas que ChangeUiLanguage() est appelé à l’accueil. Les deux instances ProductService et LanguageService ont été observées dans les arguments du constructeur de ProductController.

### Chargement des produits

1. [ProductController.Index(), ligne 21](P2FixAnAppDotNetCode/Controllers/ProductController.cs#L21) appelle _productService.GetAllProducts().
2. [ProductService.GetAllProducts(), ligne 24](P2FixAnAppDotNetCode/Models/Services/ProductService.cs#L24) appelle _productRepository.GetAllProducts() et convertit le résultat en `List<Product>`.
3. [ProductRepository.GetAllProducts(), ligne 41](P2FixAnAppDotNetCode/Models/Repositories/ProductRepository.cs#L41) sélectionne les produits dont Stock > 0, les trie par Name puis retourne un tableau.
4. La liste remonte au service puis au contrôleur ; [ProductController, ligne 22](P2FixAnAppDotNetCode/Controllers/ProductController.cs#L22) retourne View(products). Ce retour prépare un résultat MVC ; le navigateur n’a pas encore nécessairement reçu la page.

### Rendu avant l’affichage

MVC rend [Views/Product/Index.cshtml](P2FixAnAppDotNetCode/Views/Product/Index.cshtml) avec un modèle `IEnumerable<Product>`. La boucle foreach produit les lignes : nom, description, prix, stock et formulaire vers Cart/AddToCart.

Views/_ViewStart.cshtml désigne le layout _Layout. [Views/Shared/_Layout.cshtml](P2FixAnAppDotNetCode/Views/Shared/_Layout.cshtml) compose la page avec RenderBody() et les composants CartSummary et LanguageSelector.

Dans **P2FixAnAppDotNetCode.Components**, CartSummaryViewComponent(ICart) reçoit le panier ; Invoke() retourne la vue correspondante. Le modèle Cart appartient à **P2FixAnAppDotNetCode.Models**. LanguageSelectorViewComponent.Invoke(ILanguageService languageService) retourne également sa vue. L’accueil a répondu correctement lors de mon exécution ; cette description ne prétend pas tracer la résolution détaillée des arguments par MVC.

Le traitement MVC et le rendu Razor précèdent l’envoi du HTML. CartController.AddToCart et OrderController ne font pas partie du seul chargement initial des produits ; ils interviennent après les actions correspondantes.

### Choix des commandes

- **F11 — Pas à pas détaillé** : sur _productService.GetAllProducts(), entrer dans le service puis dans le dépôt pour suivre la récupération. Les symboles et l’option « Uniquement mon code » influencent l’accès au framework et à LINQ.
- **F10 — Pas à pas principal** : exécuter l’instruction et ses appels sans entrer dans leur détail ; examiner ensuite la variable products. Un point d’arrêt interne peut malgré tout interrompre l’exécution.
- **Maj+F11 — Pas à pas sortant** : terminer la méthode courante jusqu’au retour chez son appelant, du dépôt vers le service puis vers le contrôleur. Une exception ou un point d’arrêt peut interrompre ce retour.
- **F5** : continuer jusqu’au prochain arrêt.

Examiner Variables locales, Espions et Pile des appels à chaque arrêt. L’ordre effectivement observé figure dans la trace : démarrage, construction des dépôts et du service, ProductController, chargement de la liste, puis CartSummary. Au contrôleur, products passe de null à une instance `List<Product>` après le retour du service. L’accueil contient ensuite neuf produits. F11 ne traverse pas forcément automatiquement toute la chaîne asynchrone du rendu.

## 9. Exécutable Windows

Commande de publication depuis la racine du dépôt :

~~~powershell
dotnet publish P2FixAnAppDotNetCode/Diayma.csproj -c Release -r win-x64 --self-contained true -o Diayma-Mbaye-Sylla-Windows-x64
~~~

La publication a réussi et Diayma.exe a été lancé localement. GET / a répondu **HTTP 200**, avec **9 produits** et la personnalisation à mon nom. Les deux bugs ont été reproduits sur cette version publiée.

L’archive **Diayma-Mbaye-Sylla-Windows-x64.zip**, d’environ **43 Mo**, contient le runtime et les dépendances, le lanceur Lancer-Diayma.cmd, les instructions, VERIFICATION.txt et TRACE-DEBOGAGE.md. C’est une publication autonome à plusieurs fichiers : ne pas distribuer Diayma.exe seul.

Extraire toute l’archive, double-cliquer sur **Lancer-Diayma.cmd**, laisser la fenêtre ouverte et consulter **http://127.0.0.1:5080**. L’application reste un serveur web local, pas une application de bureau. Fermer la fenêtre après utilisation.

## 10. Lien Google Drive

[Télécharger l’exécutable Windows et ses dépendances](https://drive.google.com/file/d/1mo99wwJDlgVW8qtVvyFkLor5l7DQKSTy/view?usp=drivesdk).

Partage vérifié : **Tous les utilisateurs qui ont le lien — Lecteur**.

## 11. Options

L’option wolof n’est pas implémentée. Les changements sont répartis en commits : compte rendu détaillé, personnalisation de la boutique et compilation Windows par GitHub Actions. La CI vérifie la publication avant l’intégration dans develop. La remise du lien dans Google Classroom reste à effectuer.
