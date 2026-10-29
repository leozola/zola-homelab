# Meu Homelab

Repositório central de infraestrutura e serviços em containers Docker para o meu Homelab pessoal.

---

## Visão Geral

Este repositório contém os arquivos de configuração (`docker-compose.yml`) e modelos de ambiente para a gestão e deployment automatizado dos serviços do servidor. 

A arquitetura foi estruturada de forma modular, onde cada serviço possui a seu próprio diretório e arquivo compose individual.

---

## Serviços Instalados

| Serviço | Descrição | Acesso por IP | Domínio local |
| :--- | :--- | :--- | :--- |
| **[Portainer](https://www.portainer.io/)** | Gestão visual de containers e ambientes Docker | `https://<IP-DO-SERVIDOR>:9443` | `portainer.home` |
| **[Nextcloud](https://nextcloud.com/)** | Nuvem privada, gestão de ficheiros e sincronização | `http://<IP-DO-SERVIDOR>:8080` | `nuvem.home` |
| **[AIOStreams](https://github.com/viren070/aiostreams)** | Agregador de streams e conteúdos de multimédia | `http://<IP-DO-SERVIDOR>:8000` | `streams.home` |
| **[Immich](https://immich.app/)** | Backup e galeria de fotos e vídeos | `http://<IP-DO-SERVIDOR>:2283` | `fotos.home` |
| **[Pi-hole](https://pi-hole.net/)** | DNS da rede com bloqueio de anúncios | `http://<IP-DO-SERVIDOR>:8081/admin` | `pihole.home` |
| **[Nginx Proxy Manager](https://nginxproxymanager.com/)** | Proxy reverso (domínios locais e SSL) | `http://<IP-DO-SERVIDOR>:81` | `proxy.home` |

O acesso por IP:porta continua funcionando sempre. Os domínios `.home` são um caminho extra: o Pi-hole aponta todos para o servidor e o Nginx Proxy Manager encaminha cada um para o serviço certo.

---

## Estrutura de Pastas

```text
.
├── aiostreams/
│   └── compose.yml
├── immich/
│   └── compose.yml
├── nextcloud/
│   └── compose.yml
├── nginx-proxy-manager/
│   └── compose.yml
├── pihole/
│   └── compose.yml
├── portainer/
│   └── compose.yml
├── docker-compose.yml
├── .env.example
├── .gitignore
└── README.md
```

---

## Primeira Instalação (Ubuntu)

1. **Porta 53 (Pi-hole).** O Pi-hole usa a porta 53 só no `SERVER_IP`, então convive com o `systemd-resolved` do Ubuntu (que fica em `127.0.0.53`). O `SERVER_IP` do `.env` precisa ser exatamente o IP do servidor (`ip -4 addr`).
   Se ainda der `address already in use`, veja quem está usando a porta:
   ```bash
   sudo ss -tulpn | grep ':53 '
   ```
   Se for outro programa em `0.0.0.0:53` ou `*:53` (ex.: `dnsmasq`, `named`), pare-o com `sudo systemctl disable --now <serviço>`.
2. **Criar a rede compartilhada** (uma única vez):
   ```bash
   docker network create rede-server
   ```
3. **Configurar o ambiente:** `cp .env.example .env` e preencher senhas e o `SERVER_IP` (IP fixo do servidor).
4. **Subir tudo:** `docker compose up -d`
5. **DNS da rede:** teste primeiro em um aparelho, colocando o IP do servidor como DNS. Funcionando, configure no roteador (DHCP → DNS = IP do servidor).
   > Se o servidor cair, os aparelhos que usam o Pi-hole ficam sem resolver nomes da internet até ele voltar.
6. **Nginx Proxy Manager** (`http://IP:81`): troque o login inicial e crie os *Proxy Hosts*:

   | Domínio | Destino (scheme / host / porta) | Observação |
   | :--- | :--- | :--- |
   | `fotos.home` | http / `immich-server` / 2283 | Ativar *Websockets Support* |
   | `nuvem.home` | http / `nextcloud` / 80 | |
   | `streams.home` | http / `aiostreams` / 3000 | |
   | `pihole.home` | http / `pihole` / 80 | |
   | `portainer.home` | https / `portainer` / 9443 | |
   | `proxy.home` | http / `nginx-proxy-manager` / 81 | |

7. **Nextcloud pelo domínio:** autorizar o novo nome:
   ```bash
   docker compose exec -u www-data nextcloud php occ config:system:set trusted_domains 1 --value=nuvem.home
   ```

### Verificação

```bash
docker compose ps                      # todos running/healthy
nslookup fotos.home <IP-DO-SERVIDOR>   # deve retornar o IP do servidor
nslookup doubleclick.net <IP-DO-SERVIDOR>  # deve retornar 0.0.0.0 (bloqueado)
```
