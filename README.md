# AVANCE EDUCACIONAL - Plataforma de Estudos

Plataforma web em Django para organizar cursos, modulos, aulas, materiais didaticos, permissoes de acesso e progresso dos alunos.

## Arquitetura

- `avance_platform/`: configuracao principal do Django, URLs globais, ASGI/WSGI.
- `apps/accounts/`: usuario customizado, perfis de aluno, login e redirecionamento por perfil.
- `apps/courses/`: entidades academicas, uploads, permissoes de curso, configuracoes visuais e seed.
- `apps/dashboard/`: dashboard administrativo proprio para gestao de conteudo e alunos.
- `apps/student/`: portal do estudante, navegacao por trilha e progresso.
- `templates/`: telas HTML com Django Templates.
- `static/`: CSS e JavaScript da interface.
- `media/`: arquivos enviados em desenvolvimento local.

## Tecnologias

- Python 3.12+ (runtime recomendado para Railway: 3.12.8)
- Django 5.1
- PostgreSQL em producao
- SQLite como fallback local
- WhiteNoise para arquivos estaticos
- Gunicorn para servidor de producao

## Execucao local

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
pip install -r requirements.txt
Copy-Item .env.example .env
python manage.py migrate
python manage.py seed_avance
python manage.py runserver
```

URLs locais:

- Plataforma: http://127.0.0.1:8000/
- Painel admin proprio: http://127.0.0.1:8000/painel/
- Django Admin: http://127.0.0.1:8000/django-admin/

Credenciais de seed:

- Admin: `admin` / `admin12345`
- Aluno: `aluno` / `aluno12345`

## Arquivos de deploy (Railway)

- [Procfile](C:/Users/Windows/Documents/New%20project/Procfile)
- [runtime.txt](C:/Users/Windows/Documents/New%20project/runtime.txt)
- [requirements.txt](C:/Users/Windows/Documents/New%20project/requirements.txt)
- [Dockerfile](C:/Users/Windows/Documents/New%20project/Dockerfile) (tambem pronto para Railway com `$PORT`)

## Build command (Railway)

```bash
pip install -r requirements.txt && python manage.py collectstatic --noinput
```

Se o Railway usar o `Dockerfile` automaticamente, este build command pode ser ignorado (ja existe `collectstatic` no Dockerfile).

## Start command (Railway)

```bash
gunicorn avance_platform.wsgi:application --bind 0.0.0.0:$PORT --workers 3 --timeout 120 --log-file -
```

## Pre-deploy command (Railway)

```bash
python manage.py migrate --noinput
```

## Variaveis de ambiente obrigatorias (Railway)

```env
ENVIRONMENT=production
DEBUG=False
SECRET_KEY=<chave-forte>
DATABASE_URL=<fornecida-pelo-plugin-postgresql>
ALLOWED_HOSTS=<seu-servico>.up.railway.app
CSRF_TRUSTED_ORIGINS=https://<seu-servico>.up.railway.app
```

Variaveis recomendadas:

```env
RAILWAY_PUBLIC_DOMAIN=<seu-servico>.up.railway.app
SECURE_SSL_REDIRECT=True
SECURE_HSTS_SECONDS=31536000
SERVE_MEDIA_FILES=True
MEDIA_ROOT=/data/media
```

## Passo a passo de publicacao no Railway

1. Suba este projeto para GitHub ou GitLab.
2. No Railway, clique em `New Project` e conecte o repositorio.
3. Adicione um servico `PostgreSQL` no mesmo projeto.
4. No servico da aplicacao, configure as variaveis de ambiente acima.
5. Opcional recomendado para uploads persistentes: em `Service > Volumes > Add Volume`, monte em `/data`.
6. Em `Settings`, defina:
   - Build Command: `pip install -r requirements.txt && python manage.py collectstatic --noinput`
   - Pre-deploy Command: `python manage.py migrate --noinput`
   - Start Command: `gunicorn avance_platform.wsgi:application --bind 0.0.0.0:$PORT --workers 3 --timeout 120 --log-file -`
7. Faca o primeiro deploy.
8. Em `Settings > Networking > Public Networking`, clique em `Generate Domain`.
9. Atualize `ALLOWED_HOSTS`, `CSRF_TRUSTED_ORIGINS` e `RAILWAY_PUBLIC_DOMAIN` com o dominio final gerado.
10. Faca redeploy.

## Observacoes importantes para nao quebrar deploy

- Nao use SQLite em producao publica; use PostgreSQL do Railway.
- Se `SECRET_KEY` estiver vazia em producao, a app nao sobe.
- Se `ALLOWED_HOSTS` ou `CSRF_TRUSTED_ORIGINS` estiverem errados, login e formularios podem falhar.
- `migrate` precisa rodar antes do primeiro acesso funcional.
- Upload em disco local no Railway e efemero; para producao real, migre para storage externo (S3, R2 ou Cloudinary).
- `SERVE_MEDIA_FILES=True` permite servir uploads pelo Django no Railway (simples, mas nao ideal para alta escala).
