# CONTEXT.md — awsbolt

> Este arquivo existe para mitigar a perda de contexto entre sessões de IA.
> Cole o conteúdo deste arquivo no início de qualquer nova conversa com uma IA
> para que ela entenda o projeto sem precisar de re-explicações.

---

## O que é este projeto

`awsbolt` é uma ferramenta CLI para gerenciar uma instância EC2 na AWS.
É um projeto de portfólio e estudo pessoal, desenvolvido inteiramente no
GitHub Codespaces.

**Repositório:** github.com/alansmartelo/awsbolt
**Autor:** Alan Silva estudante da Escola da Nuvem (nível iniciante)

---

## Stack de tecnologias

| Camada | Tecnologia | Função |
|--------|-----------|--------|
| Infraestrutura | Terraform | IaC — estudo/portfólio (não executa no lab) |
| Operações | Rust | Binário único que chama AWS CLI via `std::process::Command` |
| Dashboard | Python + Streamlit | Interface web local (localhost) |
| Ambiente | GitHub Codespaces | Ambiente de desenvolvimento principal |

**Decisão arquitetural registrada:** O Rust usa `std::process::Command`
para chamar o AWS CLI — não usa `aws-sdk-rust` (assíncrono/Tokio).
Motivo: simplicidade para iniciante sem sacrificar o portfólio.

---

## Restrições do ambiente de laboratório

- Laboratório: Escola da Nuvem (sandbox AWS com tempo limitado)
- Região AWS: sempre `us-west-2` (Oregon)
- A cada nova sessão do laboratório, estes valores mudam e precisam
  ser reconfigurados como variáveis de ambiente no Codespace:
  - `AWS_ACCESS_KEY_ID`
  - `AWS_SECRET_ACCESS_KEY`
  - `AWS_SESSION_TOKEN`
  - `INSTANCE_ID` (formato: `i-0xxxxxxxxxxxxxxxxx`)
- Nome da instância EC2: sempre `awsbolt-lab` (criada manualmente no console)
- O Terraform existe no repositório como estudo, mas não é executado no lab

---

## Princípios de código inegociáveis

Qualquer IA que continue este projeto DEVE seguir estes princípios:

1. **Uma responsabilidade por arquivo** — cada arquivo faz uma coisa só
2. **Sem `unwrap()` em produção** — todo erro tratado com `?` ou `match`
3. **Args como lista, nunca string interpolada** — previne injeção de comando
4. **Sem credenciais no código** — sempre via variável de ambiente
5. **Sem abstrações desnecessárias** — código legível por iniciante
6. **Sem warnings no `cargo build --release`**

---

## Estrutura de arquivos do projeto

```
awsbolt/
├── .devcontainer/
│   └── devcontainer.json       # Rust + AWS CLI + Python 3.12 + Streamlit
├── infra/                      # Terraform (estudo/portfólio)
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf
├── cli/                        # Rust — binário awsbolt
│   ├── src/
│   │   ├── main.rs             # Só lê argumento e roteia
│   │   ├── config.rs           # Só lê e valida variáveis de ambiente
│   │   ├── runner.rs           # Só executa processo externo (AWS CLI)
│   │   └── commands.rs         # Só monta os args de cada comando
│   └── Cargo.toml
├── dashboard/                  # Python + Streamlit
│   ├── app.py
│   └── requirements.txt
├── .gitignore
├── CONTEXT.md                  # Este arquivo
└── README.md
```

---

## Requisitos funcionais (resumo)

### Rust CLI — comandos implementados
- `awsbolt status`  → retorna `{ state, public_ip, instance_id }` em JSON
- `awsbolt list`    → lista instâncias EC2 com ID, nome e estado em JSON
- `awsbolt start`   → liga a instância, retorna novo estado em JSON
- `awsbolt stop`    → desliga a instância, retorna novo estado em JSON
- `awsbolt --version` → exibe versão

### Saída de erro padrão
Qualquer erro retorna JSON: `{ "error": true, "message": "..." }`

### Dashboard Streamlit
- Exibe status atual (chama `awsbolt status` via subprocess)
- Exibe lista de instâncias (chama `awsbolt list`)
- Botão Ligar / Desligar com atualização automática de status
- Indicador visual de estado: verde (running), vermelho (stopped),
  amarelo (pending/stopping)
- Exibe erros legíveis quando o binário retorna JSON de erro

---

## Fases de implementação

| Fase | Status | Descrição |
|------|--------|-----------|
| Fase 1 | ✅ Concluída | AWS CLI configurado, instância EC2 criada no lab |
| Fase 2 | ⏳ Estudo | Terraform — arquivos escritos, não executados |
| Fase 3 | 🔲 Pendente | Rust — `config.rs` + `main.rs` + `awsbolt status` |
| Fase 4 | 🔲 Pendente | Rust — `list`, `start`, `stop`, tratamento de erros |
| Fase 5 | 🔲 Pendente | Dashboard Streamlit |
| Fase 6 | 🔲 Pendente | README por camada, revisão final, push |

**Atualize a coluna Status conforme for concluindo cada fase.**

---

## Como configurar o ambiente a cada nova sessão

No terminal do Codespace, após pegar as credenciais do laboratório:

```bash
export AWS_ACCESS_KEY_ID="cole_aqui"
export AWS_SECRET_ACCESS_KEY="cole_aqui"
export AWS_SESSION_TOKEN="cole_aqui"
export AWS_DEFAULT_REGION="us-west-2"
export INSTANCE_ID="cole_o_id_da_instancia_aqui"
```

Para verificar se funcionou:
```bash
aws ec2 describe-instances --instance-ids $INSTANCE_ID \
  --query 'Reservations[0].Instances[0].State.Name' \
  --output text
```
Deve retornar `running` ou `stopped`.

---

## Como continuar em nova sessão de IA

1. Abra uma nova conversa
2. Cole este arquivo inteiro como primeira mensagem
3. Diga em que fase está e o que estava fazendo
4. A IA terá contexto suficiente para continuar sem alucinar

---

*Última atualização: Fase 1 em andamento — instância EC2 sendo criada*