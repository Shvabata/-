# Върколак · Нощно село

Игра на български за 4–15 приятели. Препоръчителна компания: 7–12.
Съдържа 9 роли, случайно тайно раздаване, стаи и покани, общ чат, нощни действия, гласуване и победители.

## ВАЖНО: GitHub не е самият сървър
Качете кода в GitHub, а играта се хоства в Cloudflare Workers с D1 база. GitHub Pages не изпълнява сървъра на тази игра.
Нужни са ваши профили в GitHub и Cloudflare. Няма включени пароли, ключове или чужди хостинг настройки.
Играта още не е публикувана в интернет. След настройката по-долу ще получите HTTPS адрес.

## 1. Качване в GitHub
Най-лесно е с GitHub Desktop:
1. Инсталирайте GitHub Desktop и влезте в профила си.
2. File → Add local repository → изберете тази папка.
3. Ако пише, че папката не е Git repository, използвайте „create a repository here“, като запазите тази папка като корен.
4. Направете първи commit, после Publish repository. Може да оставите хранилището private.
5. Файлът package.json трябва да е в корена на хранилището. Включете и скритите папки .github и .openai.
Не качвайте ZIP файла като единствен файл. Не качвайте node_modules, .wrangler, .env или API токени.

## 2. Еднократно създаване на база
Инсталирайте Node.js 22 LTS или по-нова съвместима версия (с npm).
Отворете PowerShell в тази папка и изпълнете:

    npm ci
    npx wrangler login
    npx wrangler d1 create noshtno-selo-db

Влезте в Cloudflare през отворения браузър. Последната команда създава база само веднъж.
Копирайте върнатия database_id (UUID). Ако вече имате такава база, използвайте нейния идентификатор от Cloudflare dashboard.

## 3. Най-бързо публикуване от вашия компютър
Заменете ВАШИЯ-UUID с действителния database_id, без ъглови скоби:

    npm run build
    npm run cf:configure -- ВАШИЯ-UUID
    npm run cf:migrate
    npm run cf:deploy

При първо публикуване Cloudflare може да поиска да изберете workers.dev поддомейн.
Последната команда отпечатва адреса на играта. Отворете го, създайте стая и изпратете нейния линк на приятелите си.
При обновяване изпълнете отново build, cf:migrate и cf:deploy. Миграциите помнят кои промени са приложени.

## 4. Автоматично публикуване от GitHub (по желание)
Включен е .github/workflows/deploy.yml. Работи при push в main и с ръчно Run workflow.
Ако първият workflow се провали преди настройката на ключовете, това е очаквано — настройте ги и изберете Re-run jobs.

В Cloudflare създайте API token само за вашия акаунт с права:
- Account → Workers Scripts → Edit
- Account → D1 → Edit
- Account → Account Settings → Read
Използвайте шаблона Edit Cloudflare Workers, като добавите D1 Edit при нужда. Не е нужен Global API Key.

В GitHub repository → Settings → Secrets and variables → Actions:
Secrets:
- CLOUDFLARE_API_TOKEN — създаденият API token.
- CLOUDFLARE_ACCOUNT_ID — Account ID от Cloudflare.
Variables:
- CLOUDFLARE_D1_DATABASE_ID — UUID на базата.

После Actions → Publish Werewolf → Run workflow, или push към main.
Не записвайте API token в код, README, commit или чат. Пазете го само в GitHub Secrets.

## Локален преглед
След npm ci:

    npm run build
    node --import ./scripts/sites-env.mjs ./node_modules/wrangler/bin/wrangler.js d1 execute DB --local --config dist/server/wrangler.json --persist-to .wrangler/state --file drizzle/0000_material_synch.sql
    npm run dev -- --hostname 127.0.0.1

Прилагайте SQL файла само веднъж за нова локална база. Отворете показания адрес (обикновено http://127.0.0.1:5173).
Локалната база и онлайн базата са отделни. Локалният адрес не работи като интернет покана за приятелите ви.

## Проверки
При работещ локален сървър:

    node scripts/test-game.mjs
    node node_modules/typescript/bin/tsc --noEmit

## Какво има в папката
- app/ — интерфейс, правила, игрова логика и сървърни заявки.
- db/ и drizzle/ — схема и SQL за базата.
- cloudflare.json — независима конфигурация за вашия Cloudflare хостинг.
- .github/workflows/deploy.yml — автоматично публикуване.
- scripts/ и build/ — инструменти за изграждане и проверка.
- package.json и package-lock.json — зависимости и команди.
- .openai/hosting.json — декларация на необходимата DB binding за build системата, без свързан Sites проект.

Един участник на браузърен профил. За няколко играчи използвайте отделни устройства/профили.
Имената се запомнят локално; няма профили с пароли или вграден гласов чат.
Първа версия за приятелски игри: няма публичен списък със стаи или инструменти за модериране.

Официални инструкции:
https://developers.cloudflare.com/workers/get-started/guide/
https://developers.cloudflare.com/d1/reference/migrations/
https://developers.cloudflare.com/workers/ci-cd/external-cicd/github-actions/
