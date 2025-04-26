## Despliegue de Pokedex Angular en Azure Static Web Apps usando Fork

Este documento describe el flujo completo para:

1. **Configurar** tu aplicación (API y rutas de imágenes).  
2. **Añadir** cabeceras de seguridad con `staticwebapp.config.json`.  
3. **Provisionar** el recurso en el Azure Portal.  
4. **Ajustar** el CI/CD con GitHub Actions.  
5. **Verificar** el despliegue.  ([Build Static Web App on Azure with JavaScript - Learn Microsoft](https://learn.microsoft.com/en-us/azure/developer/javascript/how-to/create-static-web-app?utm_source=chatgpt.com), [How to Add Azure Active Directory B2C (AADB2C) Authentication To ...](https://learn.microsoft.com/en-us/answers/questions/1461209/how-to-add-azure-active-directory-b2c-%28aadb2c%29-aut?utm_source=chatgpt.com))

---

## Prerrequisitos

- **Suscripción activa de Azure** (por ejemplo Azure for Students con crédito disponible).  ([Build Static Web App on Azure with JavaScript - Learn Microsoft](https://learn.microsoft.com/en-us/azure/developer/javascript/how-to/create-static-web-app?utm_source=chatgpt.com))  
- **Fork** del repositorio `pokedex-angular` en tu cuenta de GitHub (no se requiere clonación local en este README).  ([How to: Deploy Fluid applications using Azure Static Web Apps](https://learn.microsoft.com/en-us/azure/azure-fluid-relay/how-tos/deploy-fluid-static-web-apps?utm_source=chatgpt.com))  
- **Azure Static Web Apps** autorizada como OAuth App en GitHub.  ([How to: Deploy Fluid applications using Azure Static Web Apps](https://learn.microsoft.com/en-us/azure/azure-fluid-relay/how-tos/deploy-fluid-static-web-apps?utm_source=chatgpt.com))  

---

## 1. Configuración del Proyecto Angular

En tu fork, edita el archivo `src/environments/environment.prod.ts` para apuntar tanto a la API como a la ruta de imágenes:

```ts
export const environment = {
  production: true,
  pokeApiUrl: 'https://pokeapi.co/api/v2/',
  imagesPath: '/assets/images'
};
```  
- `pokeApiUrl` define la URL base de la PokéAPI.  ([Create static web app with serverless app ... - Learn Microsoft](https://learn.microsoft.com/fi-fi/azure/developer/javascript/how-to/with-web-app/static-web-app-with-swa-cli?utm_source=chatgpt.com))  
- `imagesPath` es la ruta donde la Pokedex busca sus imágenes locales.  ([Create static web app with serverless app ... - Learn Microsoft](https://learn.microsoft.com/fi-fi/azure/developer/javascript/how-to/with-web-app/static-web-app-with-swa-cli?utm_source=chatgpt.com))  

Genera el build de producción con tu workflow de GitHub Actions (usado más adelante), por lo que no es necesario un paso local de build en este README.  ([Build configuration for Azure Static Web Apps - Learn Microsoft](https://learn.microsoft.com/en-us/azure/static-web-apps/build-configuration?utm_source=chatgpt.com))

---

## 2. Seguridad: `staticwebapp.config.json`

Añade en la raíz de tu fork un archivo `staticwebapp.config.json` con la siguiente configuración de seguridad y fallback para SPA:  ([Configure Azure Static Web Apps | Microsoft Learn](https://learn.microsoft.com/en-us/azure/static-web-apps/configuration?utm_source=chatgpt.com))

```json
{
  "globalHeaders": {
    "Content-Security-Policy": "default-src 'self' https://pokeapi.co; connect-src *; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; img-src * data:; font-src 'self' https://fonts.gstatic.com;",
    "X-Frame-Options": "DENY",
    "Permissions-Policy": "geolocation=(), camera=(), microphone=()",
    "Strict-Transport-Security": "max-age=63072000; includeSubDomains; preload"
  },
  "navigationFallback": {
    "rewrite": "/index.html"
  }
}
```  
- **CSP** restringe orígenes de recursos críticos.  ([Configure Azure Static Web Apps | Microsoft Learn](https://learn.microsoft.com/en-us/azure/static-web-apps/configuration?utm_source=chatgpt.com))  
- **X-Frame-Options** evita carga en iframes externos.  ([Configure Azure Static Web Apps | Microsoft Learn](https://learn.microsoft.com/en-us/azure/static-web-apps/configuration?utm_source=chatgpt.com))  
- **Permissions-Policy** bloquea APIs sensibles (geoloc, cámara, micro).  ([Configure Azure Static Web Apps | Microsoft Learn](https://learn.microsoft.com/en-us/azure/static-web-apps/configuration?utm_source=chatgpt.com))  
- **HSTS** obliga HTTPS con preload y subdominios.  ([Configure Azure Static Web Apps | Microsoft Learn](https://learn.microsoft.com/en-us/azure/static-web-apps/configuration?utm_source=chatgpt.com))  
- **navigationFallback** asegura que todas las rutas sirvan `index.html`.  ([Configuration overview for Azure Static Web Apps | Microsoft Learn](https://learn.microsoft.com/en-us/azure/static-web-apps/configuration-overview?utm_source=chatgpt.com))  

---

## 3. Provisionar Azure Static Web App

1. Entra al **Azure Portal** y selecciona **Crear un recurso** → **Static Web App**.  ([Build Static Web App on Azure with JavaScript - Learn Microsoft](https://learn.microsoft.com/en-us/azure/developer/javascript/how-to/create-static-web-app?utm_source=chatgpt.com))  
2. En **Basics**:  
   - **Subscription**: tu suscripción Azure.  
   - **Resource Group**: `Pokedex_group`.  
   - **Name**: `pokedex-app`.  
   - **Plan**: Free.  
   - **Source**: GitHub (autoriza la app si se solicita).  ([How to: Deploy Fluid applications using Azure Static Web Apps](https://learn.microsoft.com/en-us/azure/azure-fluid-relay/how-tos/deploy-fluid-static-web-apps?utm_source=chatgpt.com))  
3. En **Deployment**:  
   - **Organization**: DiegoPalmieri GitHub.  
   - **Repository**: `pokedex-angular`.  
   - **Branch**: `main` (o la rama de deploy en tu fork).  
4. En **Build Details**:  
   - **App location**: `pokedex/sistemas-distribuidos/poke-dex-lab/source/pokedex-angular`.  
   - **API location**: (vacío).  
   - **Output location**: `dist/pokedex-angular`.  ([Optimizing images with NgOptimizedImage - Angular](https://angular.io/guide/image-directive?utm_source=chatgpt.com))  
5. Haz clic en **Review + create** y luego en **Create**.  ([Build Static Web App on Azure with JavaScript - Learn Microsoft](https://learn.microsoft.com/en-us/azure/developer/javascript/how-to/create-static-web-app?utm_source=chatgpt.com))  

---

## 4. CI/CD con GitHub Actions

Azure generará automáticamente un workflow en `.github/workflows/azure-static-web-apps-*.yml` de tu fork. Ejemplo:  ([Wanted: Help understanding Sample code in Azure Samples for ...](https://learn.microsoft.com/en-us/answers/questions/1461675/wanted-help-understanding-sample-code-in-azure-sam?utm_source=chatgpt.com))

```yaml
name: Azure Static Web Apps CI/CD

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build_and_deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build and Deploy
        uses: Azure/static-web-apps-deploy@v1
        with:
          azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN }}
          repo_token: ${{ secrets.GITHUB_TOKEN }}
          action: "upload"
          app_location: "/"
          output_location: "dist/pokedex-angular"
```
- No es necesario un paso separado de `npm install && npm run build`, pues el action lo hace automáticamente.  ([Build configuration for Azure Static Web Apps - Learn Microsoft](https://learn.microsoft.com/en-us/azure/static-web-apps/build-configuration?utm_source=chatgpt.com))  
- Confirma que el secreto `AZURE_STATIC_WEB_APPS_API_TOKEN` esté configurado en tu repo.  ([Wanted: Help understanding Sample code in Azure Samples for ...](https://learn.microsoft.com/en-us/answers/questions/1461675/wanted-help-understanding-sample-code-in-azure-sam?utm_source=chatgpt.com))  

---

## 5. Verificación del Despliegue

1. En GitHub, abre la pestaña **Actions** y verifica que el workflow se complete sin errores.  ([Wanted: Help understanding Sample code in Azure Samples for ...](https://learn.microsoft.com/en-us/answers/questions/1461675/wanted-help-understanding-sample-code-in-azure-sam?utm_source=chatgpt.com))  
2. En el **Azure Portal**, selecciona tu Static Web App y copia la **URL** pública que se generó.  
3. Accede a la URL y comprueba que:  
   - La **Pokedex** carga correctamente.  
   - Las **imágenes** se sirven desde `/assets/images` según `imagesPath`.  
   - Los datos de la **PokéAPI** se consumen sin errores.  ([How to Add Azure Active Directory B2C (AADB2C) Authentication To ...](https://learn.microsoft.com/en-us/answers/questions/1461209/how-to-add-azure-active-directory-b2c-%28aadb2c%29-aut?utm_source=chatgpt.com))  

---

## Recursos Adicionales

- Documentación de **Azure Static Web Apps**: https://learn.microsoft.com/azure/static-web-apps  ([Build Static Web App on Azure with JavaScript - Learn Microsoft](https://learn.microsoft.com/en-us/azure/developer/javascript/how-to/create-static-web-app?utm_source=chatgpt.com))  
- Configuración de **staticwebapp.config.json**: https://learn.microsoft.com/azure/static-web-apps/configuration  ([Configure Azure Static Web Apps | Microsoft Learn](https://learn.microsoft.com/en-us/azure/static-web-apps/configuration?utm_source=chatgpt.com))  
- Build Configuration SWA: https://learn.microsoft.com/azure/static-web-apps/build-configuration  ([Build configuration for Azure Static Web Apps - Learn Microsoft](https://learn.microsoft.com/en-us/azure/static-web-apps/build-configuration?utm_source=chatgpt.com))  
- Angular Env Files: https://angular.io/guide/build#configure-environment-files  ([Optimizing images with NgOptimizedImage - Angular](https://angular.io/guide/image-directive?utm_source=chatgpt.com))  
- CI/CD Example: https://learn.microsoft.com/azure static-web-apps CI/CD-build-configuration  ([Optimizing images with NgOptimizedImage - Angular](https://angular.io/guide/image-directive?utm_source=chatgpt.com))
