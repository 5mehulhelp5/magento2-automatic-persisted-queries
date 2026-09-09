# Magento 2 Automatic Persisted Queries (APQ) — utrzymywany fork (SISL)

Implementacja **Automatic Persisted Queries** zgodnej z Apollo dla GraphQL w Magento 2.
Zamiast wysyłać w kółko całe zapytanie GraphQL, klient wysyła tylko jego **skrót SHA-256**;
pełne zapytanie leci do serwera raz i zostaje zapamiętane. Efekt:

- **mniejszy payload** żądań GraphQL (front wysyła hash zamiast kilobajtów query),
- **zapytania przez GET** zamiast POST → dają się cache'ować na CDN / w warstwie brzegowej,
- mniejszy narzut sieciowy dla frontów headless / PWA (Apollo Client, `apollo-link-persisted-queries`).

To **utrzymywany fork** porzuconego `danslo/magento2-module-automatic-persisted-queries`
(ostatni commit upstream: 2023). Oryginał **nie deklaruje żadnego `require`** w `composer.json`
(montuje się wszędzie, nietestowany pod nowsze wydania) i **nie zawiera `etc/acl.xml`**, mimo że
`system.xml` odwołuje się do zasobu `Danslo_Apq::config_apq` — przez co sekcja konfiguracji w
adminie była praktycznie niedostępna. Fork to naprawia i jest zweryfikowany na **2.4.9 / PHP 8.4**.

## Co poprawione w forku
- **Dodany `etc/acl.xml`** z zasobem `Danslo_Apq::config_apq` (pod `Magento_Config::config`) — sekcja *Stores → Configuration → General → Automatic Persisted Queries* jest teraz widoczna i zapisywalna. (realny bug oryginału)
- **Uzupełniony `composer.json`**: `php: ~8.1.0 || … || ~8.5.0`, `magento/framework: >=103.0.4 <104`, `magento/module-graph-ql: >=100.4.0 <101` (zamiast braku wymagań).
- Zgodność z Magento 2.4.9 potwierdzona realnym testem pełnego handshake APQ (patrz niżej).

## Zgodność
- Magento **2.4.4 – 2.4.9** (Open Source / Adobe Commerce)
- PHP **8.1 – 8.4**

## Instalacja

Paczka istnieje na Packagist, ale wskazuje na porzucony oryginał — dodaj najpierw to
repozytorium jako źródło VCS, a potem instaluj gałąź `dev-main`:

```bash
composer config repositories.sisl-apq vcs https://github.com/SISL-source/magento2-automatic-persisted-queries
composer require danslo/magento2-module-automatic-persisted-queries:dev-main
bin/magento module:enable Danslo_Apq
bin/magento setup:upgrade
bin/magento setup:di:compile        # tryb produkcyjny
bin/magento cache:enable apq         # typ cache dla przechowywanych zapytań
```

## Jak działa handshake (Apollo APQ)
1. Klient wysyła tylko `extensions.persistedQuery.sha256Hash` (bez `query`). Jeśli serwer nie zna hasha → odpowiada błędem `PersistedQueryNotFound`.
2. Klient ponawia z pełnym `query` **i** hashem. Serwer weryfikuje `sha256(query) == hash`, zapisuje zapytanie w cache `apq` i zwraca dane.
3. Kolejne żądania z samym hashem trafiają w cache i zwracają dane bez przesyłania query.

## Konfiguracja
**Stores → Configuration → General → Automatic Persisted Queries** — kody HTTP zwracane w
sytuacjach brzegowych (zgodnie z zachowaniem apollo-server):

| Pole | Domyślnie |
|------|-----------|
| HTTP code — query not found (GET) | 400 |
| HTTP code — query not found (POST) | 500 |
| HTTP code — invalid SHA (GET) | 400 |
| HTTP code — invalid SHA (POST) | 500 |

Typ cache **`apq`** (Automatic Persisted Queries) pojawia się w *System → Cache Management* —
tam też czyścisz zapamiętane zapytania.

## Licencja
MIT (jak oryginał). Fork utrzymywany przez [SISL](https://sisl.pl).
