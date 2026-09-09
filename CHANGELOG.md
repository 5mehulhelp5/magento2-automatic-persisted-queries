# Changelog

## [Unreleased] — fork SISL (2026-09)
- **Dodany `etc/acl.xml`** deklarujący `Danslo_Apq::config_apq` — `system.xml` odwoływał się do tego zasobu ACL, którego oryginał nie definiował; sekcja konfiguracji w adminie była przez to niedostępna. Teraz działa.
- **Uzupełniony `composer.json`**: `php: ~8.1.0 || ~8.2.0 || ~8.3.0 || ~8.4.0 || ~8.5.0`, `magento/framework: >=103.0.4 <104`, `magento/module-graph-ql: >=100.4.0 <101` (oryginał nie miał żadnego `require`).
- Zgodność z Magento **2.4.9 / PHP 8.4** potwierdzona pełnym testem handshake APQ (PersistedQueryNotFound → rejestracja → trafienie w cache).
- Bez zmian w logice pluginu — API i konfiguracja zgodne z oryginałem.

## Oryginał (danslo/magento2-module-automatic-persisted-queries)
- Repozytorium porzucone (ostatni commit 2023). Brak `require` w composer.json, brak acl.xml.
