# 🚀 Traefik VPS Setup

Este projeto configura o **Traefik como proxy reverso** para aplicações Docker na sua VPS, usando HTTPS automático via Let’s Encrypt.

✅ Gerencia múltiplos subdomínios
✅ Cria certificados SSL automáticos
✅ Fácil escalar novos serviços
✅ Dashboard protegido com autenticação básica

---

## 🔧 Pré-requisitos

* **Servidor VPS** com:

  * Docker
  * Docker Compose
* Domínio próprio (ex.: `meusite.com`)
* Permissões de root (ou usuário sudo)

---

## 🌐 Configuração do DNS

No seu provedor de domínio (ex.: Cloudflare, Registro.br):

| Subdomínio          | Tipo | Valor         |
| ------------------- | ---- | ------------- |
| meusite.com         | A    | IP da sua VPS |
| \*.meusite.com      | A    | IP da sua VPS |
| traefik.meusite.com | A    | IP da sua VPS |

---

## 📂 Estrutura do Projeto

```
.
├── traefik
│   └── acme.json
├── .env
└── docker-compose.yml
```

---

## ⚙️ Variáveis de Ambiente

Crie o arquivo `.env`:

```
TRAEFIK_EMAIL=seuemail@dominio.com
TRAEFIK_DOMAIN=meusite.com
TRAEFIK_HTTP_PORT=80
TRAEFIK_HTTPS_PORT=443
```

---

## 🔑 Criando acme.json

Crie o arquivo para armazenar os certificados:

```bash
mkdir traefik
touch traefik/acme.json
chmod 600 traefik/acme.json
```

---

## 🚀 Subindo o Ambiente

Suba os containers:

```bash
docker compose up -d
```

Traefik irá:

* Solicitar certificados SSL no Let’s Encrypt
* Rotejar tráfego para os serviços
* Disponibilizar dashboard seguro

---

## 🔐 Dashboard Traefik

O dashboard estará acessível em:

```
https://traefik.meusite.com/dashboard/
```

**Login básico (htpasswd):**

* Usuário: `user`
* Senha: `senha`

> **⚠️ Troque a senha no arquivo docker-compose para algo seguro.**

---

## ✅ Testando o Serviço Padrão

Acessar:

```
https://meusite.com
```

Você verá a resposta do container de teste `whoami`.

---

## ➕ Adicionando Novos Serviços

Para expor novas aplicações via Traefik, adicione no docker-compose **do seu app**:

```yaml
networks:
  web:
    external: true
```

E configure o container:

```yaml
services:
  meuapp:
    image: minha-imagem:latest
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.meuapp.rule=Host(`api.meusite.com`)"
      - "traefik.http.routers.meuapp.entrypoints=websecure"
      - "traefik.http.routers.meuapp.tls.certresolver=letsencrypt"
    networks:
      - web
```

Suba o serviço:

```bash
docker compose up -d
```

Pronto! O subdomínio estará acessível via HTTPS.

---

## 🛡️ Segurança

* Nunca exponha o dashboard sem autenticação.
* Use redes docker externas para apps separados.
* Verifique as permissões do arquivo `acme.json`.

---

## ✅ Troubleshooting

* **Erro 403 ACME Challenge**

  * Verifique se o domínio aponta para o IP da VPS.
  * Verifique se a porta 80 está liberada no firewall.

* **Dashboard não abre**

  * Verifique se o subdomínio traefik está configurado.
  * Veja logs do Traefik:

    ```
    docker logs traefik
    ```

---

## 📚 Links úteis

* [Traefik Documentation](https://doc.traefik.io/traefik/)
* [Let’s Encrypt](https://letsencrypt.org/)

