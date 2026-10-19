# 🏠 Zola Homelab

Repositório central de infraestrutura e serviços em containers Docker para o meu Homelab pessoal.

---

## 📌 Visão Geral

Este repositório contém os ficheiros de configuração (`docker-compose.yml`) e modelos de ambiente para a gestão e deployment automatizado dos serviços do servidor. 

A arquitetura foi estruturada de forma modular, onde cada serviço possui a sua própria pasta de trabalho e ficheiro de composição individual.

---

## 🛠️ Serviços Instalados

| Serviço | Descrição | Acesso Local |
| :--- | :--- | :--- |
| **[Portainer](https://www.portainer.io/)** | Gestão visual de containers e ambientes Docker | `https://<IP-DO-SERVIDOR>:9443` |
| **[Nextcloud](https://nextcloud.com/)** | Nuvem privada, gestão de ficheiros e sincronização | `http://<IP-DO-SERVIDOR>:8080` |
| **[AIOStreams](https://github.com/viren070/aiostreams)** | Agregador de streams e conteúdos de multimédia | `http://<IP-DO-SERVIDOR>:8000` |

---

## 📂 Estrutura de Pastas

```text
.
├── aiostreams/
│   └── docker-compose.yml
├── nextcloud/
│   └── docker-compose.yml
├── portainer/
│   └── docker-compose.yml
├── .env.example
├── .gitignore
└── README.md
