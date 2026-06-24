- Page 86: Replace `ng add ng-zorro-antd` with:
    - Run the command `npm install ng-zorro-antd @angular/animations -f`.
    - Create a folder named `pages` inside the `app` folder.
    - Create a folder `welcome` inside the `pages` folder.
    - Create a file named `welcome.ts` inside the `welcome` folder with the following contents:
        ```ts
        import { Component } from '@angular/core';

        @Component({
            selector: 'app-welcome',
            imports: [],
            templateUrl: './welcome.html'
        })
        export class Welcome {}
        ```
    - Create a file named `welcome.routes.ts` inside the `welcome` folder with the following contents:
        ```ts
        import { Routes } from '@angular/router';
        import { Welcome } from './welcome';

        export const WELCOME_ROUTES: Routes = [{ path: '', component: Welcome }];
        ```
    - Create a file named `welcome.html` inside the `welcome` folder with the following contents:
        ```html
        <p>welcome works!</p>
        ```
    - Open the `app.ts` file and replace its contents as follows:
        ```ts
        import { Component } from '@angular/core';
        import { RouterOutlet } from '@angular/router';
        import { NzLayoutModule } from 'ng-zorro-antd/layout';
        import { NzMenuModule } from 'ng-zorro-antd/menu';

        @Component({
            selector: 'app-root',
            imports: [RouterOutlet, NzLayoutModule, NzMenuModule],
            templateUrl: './app.html',
            styleUrl: './app.scss'
        })
        export class App {}
        ```
    - Open the `app.scss` file and add the following contents:
        ```scss
        :host {
            display: flex;
        }

        .app-layout {
            height: 100vh;
        }

        .top-nav {
            line-height: 64px;
        }

        .logo {
            float: left;
            height: 64px;
            padding-right: 24px;
            line-height: 64px;
            background: #001529;
        }

        .logo img {
            display: inline-block;
            height: 32px;
            width: 32px;
            vertical-align: middle;
        }

        .logo h1 {
            display: inline-block;
            margin: 0 0 0 15px;
            color: #fff;
            font-weight: 600;
            font-size: 20px;
            font-family:
                Avenir,
                Helvetica Neue,
                Arial,
                Helvetica,
                sans-serif;
            vertical-align: middle;
        }

        nz-content {
            padding: 24px 50px;
        }

        .inner-content {
            padding: 24px;
            background: #fff;
            height: 100%;
        }
        ```
    - Open the `app.routes.ts` file and replace its contents as follows:
        ```ts
        import { Routes } from '@angular/router';

        export const routes: Routes = [
            { path: '', pathMatch: 'full', redirectTo: '/welcome' },
            {
                path: 'welcome',
                loadChildren: () => import('./pages/welcome/welcome.routes').then((m) => m.WELCOME_ROUTES),
            },
        ];
        ```
    - Open the `app.html` file and replace its contents as follows:
        ```html
        <nz-layout class="app-layout">
            <nz-header>
                <div class="logo">
                <a>
                    <img src="https://ng.ant.design/assets/img/logo.svg" alt="logo" />
                    <h1>NG-ZORRO</h1>
                </a>
                </div>
                <ul nz-menu class="top-nav" nzTheme="dark" nzMode="horizontal">
                <li nz-menu-item routerLinkActive="ant-menu-item-selected" routerLink="/welcome">Home</li>
                <li nz-menu-item>Account</li>
                <li nz-menu-item>Profile</li>
                </ul>
            </nz-header>
            <nz-content>
                <div class="inner-content">
                <router-outlet></router-outlet>
                </div>
            </nz-content>
        </nz-layout>
        ```
    - Open the `app.config.ts` file and replace its contents as follows:
        ```ts
        import { ApplicationConfig, provideBrowserGlobalErrorListeners } from '@angular/core';
        import { provideRouter } from '@angular/router';

        import { routes } from './app.routes';
        import { en_US, provideNzI18n } from 'ng-zorro-antd/i18n';
        import { registerLocaleData } from '@angular/common';
        import en from '@angular/common/locales/en';

        registerLocaleData(en);

        export const appConfig: ApplicationConfig = {
            providers: [
                provideBrowserGlobalErrorListeners(), provideRouter(routes), provideNzI18n(en_US)
            ],
        };
        ```
    - Open the `angular.json` file and add the following in the `styles` array of the `build` section:
        ```json
            ./node_modules/ng-zorro-antd/ng-zorro-antd.min.css
        ```