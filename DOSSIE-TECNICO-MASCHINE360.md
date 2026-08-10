# Maschine360 — Dossiê técnico do sistema

Sistema de gestão de estoque de autopeças, serviços/OS e financeiro para oficinas parceiras. Rodando como app web (arquivo HTML único), com banco de dados na nuvem (Firebase Realtime Database).

## 1. Onde o sistema mora hoje

- **Hospedagem atual**: arquivo estático, publicado manualmente (ex.: Netlify "drag and drop") — não há servidor próprio, não há backend próprio.
- **Banco de dados**: Firebase Realtime Database (Google Cloud), projeto `maschine360`.
- **Autenticação**: Firebase Authentication (login por e-mail/senha).
- **Arquivos do app**: pasta com 4 arquivos — `index.html` (o sistema), `favicon.png`, `logo-icon.png`, `support.js`. Precisa ser publicada como pasta (não como HTML único) para o ícone funcionar corretamente em atalhos de celular (iOS/Android).

## 2. Credenciais Firebase atualmente em uso

```
apiKey: AIzaSyDxGfni9rBXN0ODhaxDFhrkzm6W40guXts
authDomain: maschine360.firebaseapp.com
databaseURL: https://maschine360-default-rtdb.firebaseio.com
projectId: maschine360
storageBucket: maschine360.firebasestorage.app
messagingSenderId: 407548710970
appId: 1:407548710970:web:7a7fff06d8742a99502503
```

**Importante para migração**: este projeto Firebase provavelmente está numa conta Google pessoal. Para levar para a conta empresarial, os caminhos são:
1. **Transferir a propriedade** do projeto Firebase (`maschine360`) da conta pessoal para a conta Google Workspace/empresarial (Firebase Console → Configurações do projeto → Usuários e permissões → Transferir propriedade), OU
2. **Criar um projeto Firebase novo** na conta empresarial e migrar os dados (exportar o banco atual em JSON e importar no novo projeto), atualizando as credenciais acima no código.

A opção 1 é mais simples e não exige tocar no código nem migrar dados.

## 3. Estrutura de dados (Firebase Realtime Database)

Caminho raiz: `estoque/data/`. Cada coleção é um nó separado (gravação por merge, não sobrescrita total — cada coleção salva independente das outras para evitar perda de dados em acessos simultâneos):

| Coleção | Conteúdo |
|---|---|
| `products` | Peças cadastradas (nome, categoria, montadora, unidade, estoque mínimo, veículos compatíveis) |
| `lotes` | Lotes de entrada de peças (nota fiscal, custo unitário, quantidade, restante, avulsas com TAG) |
| `saidas` | Saídas de peças (venda/uso), vinculadas a cliente, veículo e nº de OS |
| `clientes` | Prefeituras/empresas clientes (nome, categoria) |
| `veiculos` | Veículos de cada cliente (placa, modelo, ano, chassi, órgão/setor, foto) |
| `orcamentos` | Orçamentos gerados para clientes |
| `users` | Usuários do sistema (nome, e-mail, papel: admin/estoquista/supervisor, ativo/inativo, foto) |
| `servicos` | Serviços/ordens de serviço (OS, mecânico, descrição, valor, oficina, status) |
| `oficinas` | Oficinas parceiras (nome, cidade, CNPJ opcional, dados bancários/PIX) |
| `pagamentosOficina` | Histórico de pagamentos a oficinas por OS |
| `servOsPagamento` | Status de pagamento por OS+oficina (pendente/parcial/pago) |
| `servOsValorGrupo` | Valor agrupado quando serviços de uma OS são cobrados em conjunto |
| `pagamentosCliente` | Histórico de pagamentos recebidos de clientes (demonstrativos de peças) |
| `demoStatus` / `demoDocs` / `demoParcialInfo` | Status comercial dos demonstrativos de peças por OS (aguardando orçamento → NF paga) |
| `demoStatusServ` / `demoParcialInfoServ` | Mesma lógica, para demonstrativos de serviço |
| `solicitacoes` | Solicitações de compra de peça (feitas por estoquista/supervisor, aprovadas pelo admin) |
| `categoriasCustom` / `montadorasCustom` | Categorias e montadoras cadastradas via "Outros" |
| `nextId` | Contador interno de IDs |

## 4. Perfis de usuário e permissões

- **Admin**: acesso completo — peças, entradas/saídas, clientes, veículos, serviços, oficinas parceiras, financeiro, usuários, solicitações (aprovação/compra), configurações/backup.
- **Supervisor de oficina**: gestão completa de Serviços (abrir OS, mecânico, status), Oficinas parceiras, Financeiro de oficinas, além de poder dar entrada/saída de peças e enviar solicitações de compra.
- **Estoquista**: peças, entradas/saídas de estoque, solicitações de compra (sem aprovar).

## 5. Principais módulos/funcionalidades

- **Estoque de peças**: cadastro com categoria/montadora (extensíveis via "Outros"), controle FIFO de custo, alerta de estoque mínimo, alerta de peça parada (>45 dias), foto da peça (zoom/pan), veículos compatíveis por peça.
- **Entradas e saídas**: entrada com nota fiscal ou avulsa (TAG), saída vinculada a cliente/veículo/OS com busca por placa, exclusão de saída devolve ao estoque.
- **Clientes e veículos**: cadastro de veículos com placa, modelo, chassi, órgão/setor; histórico de peças e serviços por veículo; demonstrativo (fatura) de peças e de serviços com status comercial e histórico de pagamento (inclusive parcial, com nova previsão de pagamento).
- **Serviços (OS)**: numeração automática por cliente (ex.: JC0001, PG0001), múltiplos serviços por OS com valor individual ou em grupo, vínculo a oficina parceira, status de execução (aguardando peça/serviço, em execução, concluído).
- **Oficinas parceiras**: cadastro com cidade, CNPJ opcional, dados de pagamento (conta bancária/PIX), relatório de serviços executados com filtro e total, financeiro por oficina (pagamento por serviço/oficina dentro da mesma OS, não a OS inteira).
- **Solicitações de compra**: estoquista/supervisor solicita peça (existente ou nova, com foto), pode registrar cotação sem comprar; admin aprova e escolhe registrar como entrada nova ou baixa direta do estoque existente; reversão de compra/recusa possível.
- **Financeiro**: pagamentos a oficinas e de clientes, com filtro por período e status, pagamento parcial com nova data de previsão.
- **Relatórios**: relatório de solicitações por placa (com filtro de período/status, mostrar ou esconder valores), demonstrativos de peça e de serviço imprimíveis em A4 com cabeçalho da marca.
- **Configurações/Backup**: exportação total do banco em JSON (todas as coleções) para backup manual, com aviso semanal (toda sexta-feira); planejado suporte a restauração a partir desse backup.
- **Modo offline**: aviso visível quando sem conexão; dados digitados ficam em cache local até reconectar e sincronizar.

## 6. Identidade visual

- Nome do produto: **Maschine360**.
- Segmento: "Autopeças & Serviços".
- Logo: arquivo `logo-icon.png` (ícone circular).
- Tipografia: Manrope (interface), JetBrains Mono (códigos/valores monetários/placas).

## 7. O que levar para a equipe técnica/DevOps da empresa

1. Este arquivo.
2. Acesso (ou transferência de propriedade) ao projeto Firebase `maschine360`, ou decisão de criar um novo projeto Firebase na conta empresarial.
3. Domínio próprio da empresa, se desejarem trocar o link atual do Netlify por algo como `estoque.suaempresa.com.br`.
4. Pasta de deploy (`netlify-deploy/`) com os 4 arquivos do sistema.
5. Backup mais recente exportado pela tela de Configurações (JSON com todas as coleções), como rede de segurança antes de qualquer migração.
