# Guía de contribución — Paso de Lanchas

Cómo trabajamos en GitHub, paso a paso. Aplica a todos los repositorios de la organización **paso-de-lanchas**.

## 1. Preparar tu computadora (una sola vez)

1. Activa la **verificación en dos pasos** en tu cuenta de GitHub (*Settings → Password and authentication*).
2. Acepta la invitación a la organización (te llega por correo o en github.com/paso-de-lanchas).
3. Instala **Git** (git-scm.com), **Node.js LTS** (nodejs.org) y **VS Code**.
4. Configura Git con tu nombre y el correo de tu cuenta de GitHub:

   ```bash
   git config --global user.name "Tu Nombre Apellido"
   git config --global user.email "correo-de-tu-cuenta-de-github"
   git config --global pull.rebase false
   ```

5. Clona el repositorio (la primera vez que hagas `push`, Git abrirá el navegador para iniciar sesión):

   ```bash
   git clone https://github.com/paso-de-lanchas/paso-de-lanchas-app.git
   cd paso-de-lanchas-app
   ```

## 2. Ramas

| Rama | Para qué | Sale de | Regresa a |
|---|---|---|---|
| `main` | Producción: lo que usan los pasajeros | — | — |
| `develop` | Integración y pruebas (rama por defecto) | — | `main` al cierre de cada sprint |
| `feature/<issue>-descripcion` | Una historia de usuario o funcionalidad | `develop` | `develop` |
| `fix/<issue>-descripcion` | Corregir un error encontrado en pruebas | `develop` | `develop` |
| `docs/<issue>-descripcion` | Solo documentación | `develop` | `develop` |
| `chore/<issue>-descripcion` | Configuración, dependencias, CI | `develop` | `develop` |
| `test/<issue>-descripcion` | Solo pruebas | `develop` | `develop` |
| `hotfix/<issue>-descripcion` | Error grave en producción | `main` | `main` y `develop` |

Nombres en minúsculas, con guiones, sin acentos ni ñ: `feature/23-escaneo-qr-admin`.

**Nadie sube directo a `main` ni a `develop`**: GitHub lo bloquea. Todo entra por pull request.

## 3. El ciclo de cada historia

1. **Toma el issue** en el tablero: asígnatelo y muévelo a *En progreso* (máximo 2 por persona).
2. **Crea tu rama** desde `develop` actualizado:

   ```bash
   git switch develop
   git pull
   git switch -c feature/23-escaneo-qr-admin
   ```

3. **Trabaja y haz commits pequeños** (uno por cada paso lógico):

   ```bash
   git add .
   git commit -m "feat(boarding): validar código TOTP del pasajero"
   ```

4. **Sube tu rama** al menos una vez al día:

   ```bash
   git push -u origin feature/23-escaneo-qr-admin   # la primera vez
   git push                                         # las siguientes
   ```

5. **Mantén tu rama al día** si otros ya fusionaron cambios en `develop`:

   ```bash
   git fetch origin
   git merge origin/develop
   # si hay conflictos: resuélvelos en VS Code, prueba, y luego
   git add .
   git commit
   git push
   ```

6. **Abre el pull request** en GitHub (botón *Compare & pull request*):
   - Base: **develop**. Título como commit: `feat(boarding): escanear QR, cobrar y asignar lugar (#23)`.
   - Llena la plantilla y escribe `Closes #23` para que el issue se cierre solo.
   - Pide revisión a alguien **de otra área**.
7. **Atiende la revisión**: haz los cambios en la misma rama y vuelve a hacer `git push`; el PR se actualiza solo.
   Responde cada comentario y márcalo como resuelto.
8. **Fusiona** con **Squash and merge** cuando tengas la aprobación y todo esté en verde. La rama se borra sola.
9. **Limpia tu computadora**:

   ```bash
   git switch develop
   git pull
   git branch -d feature/23-escaneo-qr-admin
   ```

## 4. Mensajes de commit (Conventional Commits)

Formato: `tipo(alcance): descripción en español, en presente y sin punto final`.

| Tipo | Cuándo | Ejemplo |
|---|---|---|
| `feat` | Funcionalidad nueva | `feat(wallet): recargar saldo con Mercado Pago` |
| `fix` | Corrección de un error | `fix(boarding): evitar cobro doble al reescanear` |
| `docs` | Solo documentación | `docs(adr): registrar operación en dos muelles` |
| `test` | Pruebas | `test(trip): cubrir paso de En tránsito a Finalizado` |
| `refactor` | Mejorar código sin cambiar comportamiento | `refactor(fare): extraer tarifa de estudiante` |
| `style` | Formato | `style: aplicar prettier en admin` |
| `perf` | Rendimiento | `perf(map): consultar GPS cada 5 s` |
| `chore` | Dependencias, configuración | `chore(deps): actualizar prisma` |
| `ci` | GitHub Actions | `ci: agregar pruebas e2e` |

Alcances sugeridos: `auth`, `boarding`, `wallet`, `trip`, `fleet`, `driver`, `map`, `reports`, `admin`, `superadmin`, `pwa`, `db`, `ui`.

## 5. Pull requests y revisión

- **Una historia (o parte de ella) por PR**, idealmente menos de 400 líneas.
- **El revisor es de otra área** (Cova revisa pantallas de Diego, José revisa casos de uso de Cova…).
- **Primera revisión en máximo 24 horas.** Revisar tiene prioridad sobre empezar trabajo nuevo.
- Comenta el código, no a la persona. Prefijos: **[bloqueante]**, **[sugerencia]**, **[pregunta]**.

Lista del revisor:
- ¿Cumple los criterios de aceptación del issue?
- Saldo y cupo: el escaneo es **una sola transacción**, el QR es de un solo uso, **cero sobreventa y cero cobro doble**.
- Seguridad: Zod en toda entrada, permisos verificados en el servidor, sin contraseñas ni datos personales en el código.
- Arquitectura: las rutas API solo validan y llaman a un caso de uso; las dependencias apuntan al dominio.
- Pruebas del caso feliz y de los casos de error.

## 6. Fin de sprint y versiones

- En la Sprint Review se abre un PR **`develop` → `main`** titulado `release: v0.N.0`, con 2 aprobaciones y
  **Create a merge commit**. Luego se crea la etiqueta `v0.N.0` y un *Release* en GitHub.
- La `v1.0.0` será la primera versión en operación real.

## 7. Urgencias en producción (hotfix)

```bash
git switch main
git pull
git switch -c hotfix/52-cobro-doble
# corregir + prueba que demuestre el arreglo
git push -u origin hotfix/52-cobro-doble
```

PR hacia `main` (2 aprobaciones) → etiqueta `v0.N.1` → otro PR de `main` hacia `develop` para no perder el arreglo.

## 8. Reglas de seguridad

- Las contraseñas y llaves van **solo** en `.env.local` (Git lo ignora). En el repositorio solo existe `.env.example`.
- Nunca subas respaldos de base de datos, listas de pasajeros ni capturas con datos personales reales.
- Si subes un secreto por error, avisa de inmediato: hay que **cambiarlo**, borrar el commit no basta.
- Nunca uses `git push --force` sobre `main` o `develop` (GitHub lo bloquea).

## 9. Comandos más usados

| Quiero… | Comando |
|---|---|
| Ver en qué rama estoy y qué cambié | `git status` |
| Traer lo último de develop | `git switch develop` y `git pull` |
| Crear mi rama | `git switch -c feature/23-descripcion` |
| Guardar cambios | `git add .` y `git commit -m "feat(alcance): descripción"` |
| Subir mi rama | `git push -u origin <rama>` / `git push` |
| Actualizar mi rama con develop | `git fetch origin` y `git merge origin/develop` |
| Ver el historial | `git log --oneline --graph -15` |
| Descartar cambios de un archivo | `git restore <archivo>` |
| Guardar cambios temporalmente | `git stash` y luego `git stash pop` |
