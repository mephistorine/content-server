---
title: Конфигурация
---

Конфигурация проекта находится в файле `svelte.config.js`. Все значения необязательны. Полный список параметров со значениями по умолчанию показан здесь:

```js
/** @type {import('@sveltejs/kit').Config} */
const config = {
	// параметры для svelte.compile (https://svelte.dev/docs#svelte_compile)
	compilerOptions: null,

	// массив расширений, которые будут считаться компонентами Svelte
	extensions: ['.svelte'],

	kit: {
		adapter: null,
		amp: false,
		appDir: '_app',
		browser: {
 			hydrate: true,
 			router: true
 		},
		csp: {
 			mode: 'auto',
 			directives: {
 				'default-src': undefined
 				// ...
 			}
 		},
		files: {
			assets: 'static',
			hooks: 'src/hooks',
			lib: 'src/lib',
			routes: 'src/routes',
			serviceWorker: 'src/service-worker',
			template: 'src/app.html'
		},
		floc: false,
		inlineStyleThreshold: 0,
		methodOverride: {
 			parameter: '_method',
 			allowed: []
 		},
		package: {
 			dir: 'package',
 			// кроме файлов .d.ts и начинающихся с нижнего подчеркивания
 			exports: (filepath) => !/^_|\/_|\.d\.ts$/.test(filepath),
 			files: () => true,
 			emitTypes: true
 		},
		paths: {
			assets: '',
			base: ''
		},
		prerender: {
			concurrency: 1,
			crawl: true,
			enabled: true,
			entries: ['*'],
 			onError: 'fail'
		},
		routes: (filepath) => !/(?:(?:^_|\/_)|(?:^\.|\/\.)(?!well-known))/.test(filepath),
		serviceWorker: {
			register: true,
 			files: (filepath) => !/\.DS_Store/.test(filepath)
 		},
		trailingSlash: 'never',
		version: {
 			name: Date.now().toString(),
 			pollInterval: 0
 		},
		vite: () => ({})
	},

	// SvelteKit использует vite-plugin-svelte. Здесь можно указать его параметры.
	// Список параметров см. на https://github.com/sveltejs/vite-plugin-svelte/blob/main/docs/config.md

	// Параметры для svelte.preprocess (https://svelte.dev/docs#svelte_preprocess)
	preprocess: null
};

export default config;
```

### adapter

Требуется при запуске `svelte-kit build` и определяет, как преобразуется вывод для разных платформ. См. [Адаптеры](#adaptery).

### amp

Включить режим [AMP](#amp).

### appDir

Директория относительно `paths.assets` в которой будут находиться скомпилированные JS/CSS файлы и импортированные статические ресурсы. Имена файлов в этой директории будут содержать хеши на основе их содержимого, то есть эти файлы можно кэшировать на неопределённый срок. Значение параметра не должно начинаться или заканчиваться на `/`.

### browser

Объект, содержащий ноль или более из следующих значений типа `boolean`:
- `hydrate` — следует ли [гидрировать](#page-options-hydrate) серверный HTML с помощью клиентского приложения. Устанавливайте этот параметр в `false` только в исключительных случаях.
- `router` - включает или отключает клиентский [роутер](#page-options-router) по всему приложению.

### csp

Объект, содержащий ноль или более из следующих значений:

- `mode` - 'hash', 'nonce' или 'auto'
- `directives` — объект, содержащий пары `[директива]: значения[]`.

Настройка [Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy). CSP помогает защитить пользователей от XSS-атак, ограничивая места, из которых могут загружаться ресурсы. Например, такая конфигурация...

```js
 {
 	directives: {
 		'script-src': ['self']
 	}
 }
 ```

...позволит предотвратить загрузку скриптов с внешних сайтов. Для любых инлайновых стилей и генерируемых скриптов SvelteKit автоматически дополнит указанные директивы нужным хешем(в зависимости от `mode`).

При предварительной отрисовке страниц заголовок CSP добавляется с помощью тега `<meta http-equiv>` (учтите, что в этом случае директивы `frame-ancestors`, `report-uri` и `sandbox` будут игнорироваться).

> При значении `mode: 'auto'`, SvelteKit будет использовать аттрибуты nonce для динамических страниц и аттрибуты hashe для предварительно отрисованных. Использование аттрибута nonce с предварительно отрисованными страницами небезопасно а, значит, запретно.

### files

Объект, содержащий следующие строковые параметры(все опциональны):

- `assets` — место для размещения статических файлов, которые должны иметь стабильные URL-адреса и не подвергаться обработке, например `favicon.ico` и `manifest.json`
- `hooks` — путь к файлу хуков (см. [Хуки](#huki))
- `lib` — внутренняя библиотека вашего приложения, доступная во всей кодовой базе как `$lib`
- `routes` — файлы, которые определяют структуру вашего приложения ([Маршруты](#marshruty))
- `serviceWorker` — файл [сервис-воркера](#servis-vorkery)
- `template` — расположение шаблона для HTML-ответов сервера


### floc

Google [FLoC](https://github.com/WICG/floc) — это технология таргетированной рекламы, которую [Electronic Frontier Foundation](https://www.eff.org/) сочла [вредной](https://www.eff.org/deeplinks/2021/03/googles-floc-terrible-idea) для конфиденциальности пользователей. [Браузеры, отличные от Chrome](https://www.theverge.com/2021/4/16/22387492/google-floc-ad-tech-privacy-browsers-brave-vivaldi-edge-mozilla-chrome-safari) отказались её реализовать.

SvelteKit по умолчанию отключает использование FLoC в приложении, также как это делают другие сервисы, например [GitHub Pages](https://github.blog/changelog/2021-04-27-github-pages-permissions-policy-interest-cohort-header-added-to-all-pages-sites/ ). Если для параметра `floc` не установлено значение `true`, к ответу сервера будет добавлен такой заголовок:

``` 
Permissions-Policy: Interest-cohort = () 
```

> Это может применяться только при серверном рендеринге — заголовки для предварительно отрисованных страниц (например, созданных с помощью [adapter-static](https://github.com/sveltejs/kit/tree/master/packages/adapter-static)) определяются платформой хостинга.

### inlineStyleThreshold

Встраивание CSS-стилей на странице внутри блока `<style>` в теге `<head>`. Параметр указывает максимальный размер файла, который будет встроен в страницу. Содержимое всех CSS-файлов, которые необходимы данной странице и по размеру меньше указанного значения, будет слито и помещено в блок `<style>`.

> Установка данного параметра приведет к меньшему количеству первоначальных запросов и может улучшить метрику [Первой Отрисовки Контента](https://web.dev/i18n/ru/fcp/). Но при этом получаемый HTML возрастает в размере, а эффективность кэширования в браузере снижается.

### methodOverride

См. [HTTP методы](#marshruty-endpointy-http-metody). Объект, содержащий ноль или более из следующего:

- `parameter` - имя параметра запроса, используемое для передачи требуемого значения метода
- `allowed` - массив методов HTTP, которые могут быть использованы при переопределении исходного метода запроса

### package

Опции, связанные с [созданием пакетов](#sozdanie-paketov).

- `dir` - директория для готового пакета
- `emitTypes` - по умолчанию `svelte-kit package` автоматически сгенерирует файлы определений типов `.d.ts` для пакета. Хоть этот параметр и можно отключить, но мы считаем, что для качества экосистемы  нужно всегда генерировать файлы определений типов. Убедитесь, что есть веская причина устанавливать значение равным `false` (например, если имеются определения типов, написанные вручную).
- `exports` - функция вида `(filepath: string) => boolean`. Когда возвращает `true`, путь к файлу будет включён в поле `exports` в `package.json`. Любые существующие значения в исходном `package.json` будут объединены с полученными из `exports` значениями
 - `files` - функция вида `(filepath: string) => boolean`. Когда возвращает  `true`, файл по указанному пути будет обработан и скопирован в папку, определенной в параметре `dir`

 Для удобного сопоставления путей в функциях значений `exports` и `files`, можно использовать различные внешние библиотеки:

 ```js
 // svelte.config.js
 import mm from 'micromatch';

 export default {
 	kit: {
 		package: {
 			exports: (filepath) => {
 				if (filepath.endsWith('.d.ts')) return false;
 				return mm.isMatch(filepath, ['!**/_*', '!**/internal/**'])
 			},
 			files: mm.matcher('!**/build.*')
 		}
 	}
 };
 ```

### paths

Объект, содержащий следующие строковые параметры(все опциональны):

- `assets` — абсолютный путь, откуда будут загружаться статические файлы приложения. Это может быть полезно в случае, когда эти файлы находятся в каком-либо внешнем хранилище.
- `base` — путь относительно корня, который должен начинаться, но не заканчиваться на `/` (например, `/base-path`). Указывает, где будет находиться приложение и позволяет запускать приложение из поддиректории основного домена.


### prerender

См. [Предварительная отрисовка](#parametry-straniczy-prerender). Объект, содержащий следующие параметры(все опциональны):

- `concurrency` - сколько страниц может отрисовываться одновременно. Хоть JS и однопоточный, но в случаях, когда производительность отрисовки зависит от сети (например, загрузка контента с удалённой CMS), это поможет ускорить процесс, поскольку во время ожидания ответа сети будут обрабатываться другие задачи.
- `crawl` — определяет, должен ли SvelteKit находить страницы для предварительной отрисовки, переходя по ссылкам с исходных страниц
- `enabled` — установите в `false`, чтобы полностью отключить предварительную отрисовку
- `entries` — массив страниц для предварительной отрисовки или начала сканирования (при `crawl: true`). Строка `*` включает все нединамические маршруты (т.е. страницы без `[параметр]`)
- `onError`

   - `'fail'` — (по умолчанию) прерывает сборку при обнаружении ошибки маршрутизации при переходе по ссылке
   - `'continue'` — позволяет продолжить сборку, несмотря на ошибки маршрутизации
   - `function` — пользовательский обработчик ошибок, в котором можно вывести сообщение, вызвать исключение или прервать сборку, а также совершить любые другие действия, основываясь на полученных аргументах

     ```ts
	 import static from '@sveltejs/adapter-static';

     /** @type {import('@sveltejs/kit').PrerenderErrorHandler} */

     const handleError = ({ status, path, referrer, referenceType }) => {
     	if (path.startsWith('/blog')) throw new Error('Missing a blog page!');
     	console.warn(`${status} ${path}${referrer ? ` (${referenceType} from ${referrer})` : ''}`);
     };

     export default {
     	kit: {
     		adapter: static(),
     		prerender: {
     			onError: handleError
     		}
     	}
     };
     ```

### routes

Функция вида `(filepath: string) => boolean`, которая определяет, какие файлы создают маршруты, а какие рассматриваются как [приватные модули](#routing-private-modules).

### serviceWorker

Объект, содержащий ноль или более из следующих значений:

- `register` - если установлено значение `false`, отключит автоматическую регистрацию сервис-воркера
- `files` - функция вида `(filepath: string) => boolean`. Когда возвращает `true`, запрошенный файл будет доступен в  `$service-worker.files`.

### trailingSlash

 Следует ли удалять, добавлять или игнорировать конечные слэши при разрешении URL-адресов в маршруты.

 - `"never"` - перенаправит `/x/` на `/x`
 - `"always"` - перенаправит `/x` на `/x/`
 - `"ignore"`- ничего не делать. `/x` и `/x/` будут рассматриваться эквивалентно

Этот параметр влияет также и на [предварительную отрисовку](/docs#parametry-straniczy-prerender). Для соответствия с работой web-серверов при сохранении отрисованной страницы с адресом `/about`, когда параметр `trailingSlash` имеет значение `always`, будет создана директория с индексным файлом `about/index.html`, в иных случаях просто файл `about.html`.

> Игнорирование конечных слэшей не рекомендуется — семантика относительных путей в двух случаях различается (`./y` от `/x` будет `/y`, а от `/x/` станет `/x/y`). Кроме того, `/x` и `/x/` рассматриваются как отдельные URL-адреса, что вредно для SEO. При использовании этой опции, убедитесь, что логика добавления или удаления конечных слэшей из `request.path` реализована в функции [`handle`](/docs#huki-handle).

### version

Объект, содержащий ноль или более из следующих значений:

- `name` - текущая весрия приложения в виде строки
- `pollInterval` - интервал в миллисекундах для опроса новой версии

Навигация на стороне клиента может сбоить, когда на сервер загружена новая версия приложения, а люди всё ещё используют ранее загруженную версию. Если данные для следующей страницы уже загружены с сервера, они могут быть устаревшими. В таком случае может быть запрошен более не существующий JavaScript-файл. SvelteKit решает эту проблему, отключая клиентский роутер и выполняя переход с перезагрузкой страницы, когда обнаруживает, что значение версии приложения, указанное в `name`,  изменилось (по умолчанию в качестве версии используется метка времени сборки).

При установке значения `pollInterval` отличным от ноля, SvelteKit будет периодически опрашивать сервер на наличие новой версии и установит значение хранилища [`updated`](#moduli-$app-stores) в `true`, когда она будет обнаружена.

### vite

[Объект конфигурации Vite](https://vitejs.dev/config) или функция, которая его возвращает. Вы можете использовать [плагины Vite и Rollup](https://github.com/vitejs/awesome-vite#plugins) в параметре [`plugins`](https://vitejs.dev/config/#plugins)для более продвинутой сборки приложения,например, использовав оптимизацию изображений, Tauri, WASM, Workbox и прочее. Некоторые параметры сборки установить невозможно, они используются внутри SvelteKit и определяются его параметрами.