# Специфікація міграції: Webflow → Astro

## 1. Мета
Відтворити наявний сайт (експортований з Webflow) 1-в-1 візуально та функціонально на базі Astro.js,
без Next.js-патернів, без TypeScript, на функціональних компонентах/чистому JS.

- Оригінальний сайт (Webflow, опубліковано): https://spraga.webflow.io/
- Домен продакшену: __________
- Хостинг (Vercel / Netlify / інше): __________

### Обсяг робіт
Переносимо **лише головну сторінку та системні сторінки, наявні в `webflow-export/`**:
- `index.html` — головна
- `404.html` — сторінка помилки
- `styleguide.html` — стайлгайд (за потреби)

Інші сторінки оригінального сайту (якщо є) — поза скоупом цієї міграції.

## 2. Розташування вихідних файлів
Сирі файли, експортовані з Webflow, лежать у `webflow-export/` — Клод має читати їх
напряму як референс (HTML/CSS/JS), а не тільки цю специфікацію.

## 3. Карта сайту
| URL | Файл у webflow-export | Опис | Пріоритет | Статус |
|---|---|---|---|---|
| / | index.html | Головна | 1 | ✅ Перенесено у `src/pages/index.astro`, звірено скріншотом з оригіналом |
| /404 | 404.html | Сторінка помилки | 2 | Не почато |
| /styleguide | styleguide.html | Стайлгайд (системна) | 3 | Не почато |

## 4. Спільні частини (layout)
- Реалізовано в [`src/layouts/BaseLayout.astro`](../src/layouts/BaseLayout.astro): head/meta (title, description, OG, Twitter — через пропси), favicons, підключення всіх 4 CSS-файлів, глобальні скрипти (touch-detect, jQuery, webflow.js).
- Header (nav) і Footer наразі живуть прямо всередині `index.html`/`index.astro` (не винесені в окремі `.astro`-компоненти) — винести в `src/components/Nav.astro` і `Footer.astro` при переносі 404/styleguide, якщо вони повторюються на цих сторінках.
- Підключені шрифти: Inter (Regular 400/Medium 500/SemiBold 600), Seriguela (Bold 700/Black 900) — `@font-face` у `lumos-components.css`, файли в `public/fonts/`.
- Favicon / meta: реальні favicon-файли сайту підключені (`/images/favicon*.jpg`, `/images/webclip*.jpg`); title/description лишаються заглушками з оригінального експорту ("Website Name" / "Place website meta description here") — **потрібен реальний контент від користувача**.

## 5. Компоненти (повторювані блоки)
| Компонент | Де використовується | Примітки |
|---|---|---|
| Hero | | |
| Картка (card) | | |
| Форма | | |

## 6. Дизайн-токени
Усі CSS-змінні (кольори, типографіка, spacing, radius, сітка тощо) перенесені з
`webflow-export/css/spraga.webflow.css` (`:root`) у [`src/styles/global-styles.css`](../src/styles/global-styles.css) — підключити цей файл глобально в `BaseLayout.astro`.

Базові структурні класи Webflow (`.w-nav`, `.w-container`, `.w-form`, `.w-dropdown`, `.w-richtext`,
`.w-background-video` тощо — лише ті, що реально використовуються на index/404/styleguide, без
невживаних підсистем grid/slider/tabs/lightbox) перенесені у [`src/styles/webflow-base.css`](../src/styles/webflow-base.css) — теж підключити глобально в `BaseLayout.astro`.

Кастомні класи Lumos-фреймворку (page_wrap, nav_*, hero_*, card_item тощо — лише ті, що реально
використовуються на `index.html`) разом з `@font-face` для Inter/Seriguela перенесені у
[`src/styles/lumos-components.css`](../src/styles/lumos-components.css) — теж підключити глобально в `BaseLayout.astro`.
Файли шрифтів скопійовано в `public/fonts/`.

Той самий файл містить окрему секцію з явними `#w-node-<id> { grid-area: ... }` правилами —
**це критично**: Webflow/Lumos розміщує елементи в CSS Grid не лише класами, а й через
згенеровані ID конкретних елементів (`id="w-node-..."`). Без цих правил grid-елементи
"схлопуються" (0 розміру/невірна колонка) — так і сталось із відео в секції `cta_visual_wr`,
поки правило не було додане. При переносі НАСТУПНИХ сторінок (404, styleguide) обов'язково
перевіряти `spraga.webflow.css` на такі `#w-node-*` правила для ID, що зустрічаються в
відповідному HTML, а не тільки клас-орієнтовані стилі.

Глобальний inline-CSS "рушій" Lumos (box-sizing reset, fluid root font-size, container-query
based responsive grid, кілька дрібних утиліт — однаковий на всіх сторінках, був вбудований
через `<style>` в `<body>` кожної сторінки) перенесено у [`src/styles/lumos-engine.css`](../src/styles/lumos-engine.css) — підключити глобально в `BaseLayout.astro`.

## 7. Динамічний контент / CMS
- Чи є Webflow CMS-колекції (блог, кейси, товари тощо)? __________
- Якщо так — джерело даних після міграції (статичний JSON/Markdown у content collections, чи Supabase): __________

## 8. Форми та інтеграції
| Форма | Сторінка | Поля | Куди відправляти дані після міграції |
|---|---|---|---|
| | | | Supabase (/lib/supabase.js) |

## 9. Анімації / інтерактивність
Знайдено і перенесено (як inline `<style>`/`<script>` на самій сторінці, `is:inline` для скриптів):
| Елемент | Поведінка | Реалізація |
|---|---|---|
| `.marquee_row` (бігуча стрічка "KOMBUCHA") | Безкінечна горизонтальна прокрутка, пауза на hover | CSS `@keyframes marquee-to-left`, різна дистанція на мобільному |
| `.products_item` (картки продуктів) | На hover: зображення трохи збільшується і зсувається, підпис проявляється | CSS `:hover` transitions |
| `.button_main_icon` (стрілка в кнопках) | На hover іконка обертається (кут залежить від варіанту кнопки: primary/back/yellow) | CSS через `[data-wf--button-main--variant]` |
| `.ingredients_item_wr` (секція "LE GOÛT AVANT TOUT") | Fade-in + зсув знизу вгору, стагером, при досягненні низу секції в’юпорту | GSAP + ScrollTrigger (CDN), інлайн-скрипт в `index.astro` |
| `.form_fieldset_list` (чекбокси у формах) | Перемикання flex-напрямку на вузьких контейнерах | CSS `@container` запити |
| nav-гамбургер, dropdown, форми | Відкриття/закриття, валідація, locale-перемикач | Через сам `js/webflow.js` runtime (підключено як є, не переписано) |

Все інше (nav, dropdown, форми) покладається на оригінальний `webflow.js` runtime, підключений у `BaseLayout.astro` — нативна Webflow-логіка працює "як є" завдяки збереженим `data-*` атрибутам у розмітці.

## 10. Сторонні скрипти (з <head> / перед </body> оригіналу)
- Аналітика: __________
- Піксель/реклама: __________
- Чат-віджет: __________
- Інше: __________

## 11. SEO / доступність
- Чи треба зберегти існуючі URL (редиректи для SEO)? __________
- Мова(и) сайту: __________
- Особливі вимоги до accessibility: __________

## 12. Сторінки, які НЕ переносимо
- __________

## 13. Порядок робіт (для Клода)
1. ~~Створити `BaseLayout.astro` на основі спільних head/header/footer.~~ ✅
2. ~~Перенести головну сторінку (`/`) повністю, звірити з оригіналом.~~ ✅ (звірено скріншотами headless Chrome проти `https://spraga.webflow.io/` — макет, кольори, шрифти, hover/marquee/gsap-анімації збігаються)
3. Винести повторювані блоки (nav, footer, кнопки) в компоненти `src/components/`.
4. Перенести решту сторінок за пріоритетом з розділу 3 (404, styleguide) — **не забути перевірити `#w-node-*` grid-правила для кожної нової сторінки** (див. розділ 6).
5. Підключити форми до реального бекенду (Supabase).
6. Замінити заглушки title/description/OG на реальний контент.
7. Фінальна звірка: адаптивність (мобільна верстка), доступність, SEO-теги.
