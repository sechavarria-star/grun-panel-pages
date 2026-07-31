# Panel Grunwaldt — appgrun.giwa-ia.com

Panel de KPIs y transcripciones del agente de Grunwaldt, servido como **sitio
estático** bajo dominio propio. Es la migración de `CODE/grun-panel`, que era
una web app de Apps Script.

## Por qué se sacó de Apps Script

No hay forma de apuntar un CNAME a un `/exec`, y proxear la URL tampoco sirve:
el `/exec` no devuelve la página, devuelve un wrapper que adentro mete un iframe
a `script.googleusercontent.com`, y `google.script.run` habla entre esos dos
orígenes por `postMessage`. La única salida sin iframe es que la página no sea
de Apps Script.

Se pudo hacer porque el Apps Script de este panel **no hacía nada propio**: solo
guardaba la clave y reenviaba al webhook de n8n. Esa validación se movió a n8n y
Apps Script desapareció del circuito.

```
appgrun.giwa-ia.com  →  GitHub Pages (este index.html)
                             ↓ fetch ?k=<clave>
                        n8n "Grunwaldt - API Dashboard" (UK6bfoTDe2Cr5Mr7)
                             ↓
                        Supabase + planilla + calendario
```

## La clave

La clave que tipea el cliente **es** la credencial de la API: viaja como `?k=` y
la valida el nodo `Puerta` del workflow, que está pegado al Webhook a propósito
— la URL es pública, así que un pedido sin clave no llega a tocar Google Sheets,
el calendario ni Postgres. No hay ningún secreto escondido en este archivo: no
habría dónde esconderlo, es HTML estático.

Para cambiar la clave se edita la constante `CLAVES` de ese nodo. La lista
todavía acepta el viejo secreto del `/exec` (`gr7k2Qm9XvB4tLp6`) para que la web
app anterior siga andando durante la transición: **sacarlo cuando este panel
esté en producción**.

El freno anti-fuerza-bruta que hacía `Utilities.sleep(800)` ahora es el nodo
`Frenar` (Wait de 1s) en la rama de rechazo.

## CORS

El nodo Webhook declara `allowedOrigins = https://appgrun.giwa-ia.com`. Un GET
sin headers propios no dispara preflight, por eso `pedirDatos()` no manda
ninguno — si se le agrega un header, hay que habilitar el preflight.

Para probar en local hay que **agregar temporalmente** el origen del server de
pruebas (`http://localhost:8080`) a esa lista, y sacarlo después.

```bash
python3 -m http.server 8080 --directory .
```

## Deploy

Repo propio con GitHub Pages, branch `main` /root, custom domain
`appgrun.giwa-ia.com` (archivo `CNAME`). El DNS del dominio se administra en
**Squarespace**: hace falta un `CNAME appgrun → sechavarria-star.github.io`.
El apex y `www` van a Google Sites — no tocar.

Si el certificado se traba en "None", el truco conocido es quitar y re-agregar
el dominio (ver `app-giwa`).
