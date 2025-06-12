# Análise e Planejamento Futuro - Sistema de Monitoria IC

## 1. Introdução

O Sistema de Monitoria IC é uma aplicação web abrangente para gerenciar todo o fluxo de trabalho do programa de monitoria da UFBA, desde a proposta de projetos pelos professores até a seleção e cadastro final dos monitores.

**Estado Atual (Atualizado - Janeiro 2025):** O sistema possui aproximadamente 95% das funcionalidades implementadas, com uma arquitetura sólida baseada em TanStack Start, PostgreSQL/Drizzle ORM, e autenticação via CAS/UFBA. Os módulos 1, 2 e 3 estão completos, e módulo 4 foi totalmente implementado.

**Objetivo deste Documento:** Servir como guia detalhado para os próximos passos do desenvolvimento, identificando lacunas funcionais e propondo implementações estruturadas seguindo as melhores práticas do projeto.

## 2. Análise do Sistema Atual e Requisitos dos Clientes

### Principais Funcionalidades Identificadas

Com base na transcrição da reunião (`videoplayback.txt`) e tickets preliminares (`tickets.txt`), o sistema deve gerenciar:

1. **Fluxo de Projetos de Monitoria**
   - Importação do planejamento semestral
   - Geração automática de projetos pré-preenchidos
   - Assinatura digital de documentos (via `react-signature-canvas`)
   - Exportação para PROGRAD

2. **Processo Seletivo**
   - Período de inscrições com edital interno
   - Aplicação online de alunos
   - Seleção por professores com cálculo de notas
   - Geração de atas e resultados

3. **Cadastro de Monitores**
   - Aceite/recusa de vagas
   - Limite de 1 bolsa por aluno/semestre
   - Termos de compromisso (assinatura digital via `react-signature-canvas`)
   - Planilhas finais para PROGRAD

### Mapeamento para Entidades do Banco

O schema atual (`src/server/database/schema.ts`) suporta bem os requisitos:

- **projeto**: Gerencia projetos com workflow de status
- **inscricao**: Controla aplicações de alunos
- **vaga**: Registra monitores aceitos
- **periodo_inscricao**: Define janelas de inscrição
- **projeto_documento**: Armazena documentos (PDFs gerados, metadados de assinatura)
- **assinatura_documento** (Nova sugestão): Tabela para armazenar dados da assinatura digital (e.g., imagem base64, timestamp, userId) vinculada a um `projeto_documento` ou `termo_compromisso`.
- **inscricaoDocumentoTable**: Armazena documentos específicos de cada inscrição.

### Arquitetura e Suporte aos Requisitos

A arquitetura atual baseada em:
- **TanStack Start**: Oferece roteamento type-safe e SSR
- **Drizzle ORM**: Queries type-safe e migrações
- **MinIO**: Armazenamento seguro de documentos (PDFs base gerados)
- **Lucia Auth + CAS**: Autenticação integrada com UFBA
- **react-signature-canvas**: Para captura de assinaturas digitais no frontend.

Está bem preparada para expansão, necessitando principalmente de novas features e endpoints.

## 3. Funcionalidades Pendentes, Melhorias e Sugestões de Implementação

### Módulo 1: Gestão de Projetos de Monitoria (Professores e Admin)

#### 1.1 Importação de Planejamento Semestral

**Status Atual:** ✅ **IMPLEMENTADO**

**Requisitos do Cliente:**
- Importar planilha Excel com planejamento do semestre
- Criar projetos automaticamente para cada disciplina
- Pré-preencher com dados históricos

**Funcionalidades Implementadas:**
- ✅ Endpoint em `/api/projeto/import-planning` que processa arquivos .xlsx.
- ✅ Validação de dados da planilha com Zod.
- ✅ Criação de projetos em lote com status `PENDING_PROFESSOR_SIGNATURE`.
- ✅ Utilização de `projeto_template` para pré-preenchimento de dados.
- ✅ Interface de importação para admin em `/home/admin/import-projects` com drag-and-drop.
- ✅ Hook `useProjectImport` para gerenciar o upload e o estado da UI.
- ✅ Feedback de sucesso/erro via toasts.

#### 1.2 Geração Automática de PDF de Projetos

**Status Atual:** ✅ **IMPLEMENTADO** (geração completa com assinatura digital)

**Funcionalidades Implementadas:**
- ✅ Template PDF profissional com `@react-pdf/renderer`
- ✅ Geração server-side via endpoint `/api/projeto/$id/pdf`
- ✅ Campos de assinatura digital integrados
- ✅ Preenchimento automático de datas (aprovação, assinatura)
- ✅ Download direto pelo professor e admin

#### 1.3 Fluxo de Assinatura pelo Professor

**Status Atual:** ✅ **IMPLEMENTADO** (assinatura digital integrada)

**Funcionalidades Implementadas:**
- ✅ Professor assina digitalmente via `react-signature-canvas`
- ✅ Sistema atualiza status DRAFT → SUBMITTED automaticamente  
- ✅ Notificação automática para todos os admins
- ✅ Interface PDF interativa com preenchimento de datas

#### 1.4 Sistema de Assinatura Digital Unificado

**Status Atual:** ✅ **IMPLEMENTADO** (fluxo completo integrado)

**Funcionalidades Implementadas:**
- ✅ Endpoint unificado `/api/projeto/$id/assinatura` (professor e admin)
- ✅ Interface de assinatura digital com `react-signature-canvas`
- ✅ Auto-preenchimento de datas (aprovação e assinatura)
- ✅ Fluxo automático: Professor → Admin → Notificações
- ✅ Interface administrativa para assinatura (`/home/admin/document-signing`)

#### 1.5 Sistema de Notificações por Email

**Status Atual:** ✅ **IMPLEMENTADO** (integrado ao fluxo de assinatura)

**Funcionalidades Implementadas:**
- ✅ Notificação automática para admins quando professor submete
- ✅ Notificação automática para professor quando admin aprova
- ✅ Templates personalizados por contexto
- ✅ Integração com `emailService` existente

**Pendências/Melhorias:**
- [ ] Lembretes automáticos para assinaturas pendentes
- [ ] Histórico de notificações enviadas
- [ ] Templates HTML mais elaborados

#### 1.6 Geração de Planilha PROGRAD

**Status Atual:** ✅ **IMPLEMENTADO**

**Requisitos do Cliente:**
- Exportar projetos aprovados no formato PROGRAD
- Incluir todos os campos obrigatórios
- Permitir download pelo admin

**Funcionalidades Implementadas:**
- ✅ Endpoint `/api/relatorios/planilhas-prograd` refatorado para usar `exceljs`.
- ✅ Geração de planilhas .xlsx com abas separadas para "Projetos Aprovados" e "Monitores Selecionados".
- ✅ Cabeçalhos estilizados e colunas com largura definida para melhor legibilidade.
- ✅ Hook `useProgradExport` que gerencia o download e permite filtros por ano, semestre e departamento.
- ✅ Interface para admin em `/home/admin/relatorios` para selecionar filtros e baixar o relatório.
- ✅ Nomenclatura dinâmica de arquivos com base nos filtros selecionados.

### Módulo 2: Edital Interno e Inscrições (Admin e Alunos)

#### 2.1 Interface de Distribuição de Bolsas

**Status Atual:** ✅ **IMPLEMENTADO**

**Requisitos do Cliente:**
- Admin define quantidade de bolsas após retorno PROGRAD
- Distribuição por projeto/disciplina
- Visualização consolidada

**Funcionalidades Implementadas:**
- ✅ Campo `bolsasDisponibilizadas` na tabela `projeto` para armazenar alocações
- ✅ Página `/home/admin/scholarship-allocation` com interface completa para distribuir bolsas
- ✅ Endpoint `/api/projeto/$id/allocate-scholarships` com validação de admin e controle de acesso
- ✅ Hook `useScholarshipAllocation` para gerenciar estado da UI e invalidação de queries
- ✅ Tabela interativa mostrando projetos aprovados com inputs numéricos para definir bolsas
- ✅ Validação de dados com Zod e feedback de sucesso/erro via toasts
- ✅ Interface responsiva com loading states e controle de permissões

#### 2.2 Geração de Edital Interno

**Status Atual:** ✅ **IMPLEMENTADO**

**Requisitos do Cliente:**
- Gerar PDF do edital com todas as vagas
- Incluir regras e prazos
- Publicação automática

**Funcionalidades Implementadas:**
- ✅ Tabela `editalTable` no schema com todos os campos necessários
- ✅ Endpoint `/api/edital/generate` para gerar editais com validação de admin
- ✅ Template profissional `EditalTemplate` em `src/server/lib/pdfTemplates/edital.tsx`
- ✅ Interface completa em `/home/admin/edital-management` para gerenciar editais
- ✅ Agregação automática de projetos aprovados por período
- ✅ Geração de PDF com informações de vagas, datas e regras
- ✅ Sistema de publicação/despublicação de editais
- ✅ Download de editais gerados
- ✅ Armazenamento seguro no MinIO com nomenclatura organizada
- ✅ Build funcionando sem erros de lint

#### 2.3 Validação de Documentos Obrigatórios por Inscrição

**Status Atual:** ✅ **IMPLEMENTADO**

**Requisitos do Cliente:**
- Alunos devem enviar documentos para cada inscrição.
- O sistema deve validar os documentos obrigatórios.
- Bloquear inscrições com documentos pendentes.

**Funcionalidades Implementadas:**
- ✅ Tabela `inscricaoDocumentoTable` para associar arquivos a uma inscrição específica.
- ✅ O endpoint `/api/inscricao/createInscricao` foi atualizado para receber um array de documentos.
- ✅ A UI no modal de inscrição (`/home/student/vagas/page.tsx`) agora inclui uploaders para os documentos necessários.
- ✅ A lógica de negócios no frontend desabilita o botão de inscrição até que todos os documentos obrigatórios sejam carregados.

### Módulo 3: Seleção de Monitores e Atas (Professores e Admin)

#### 3.1 Sistema de Avaliação com Notas

**Status Atual:** ✅ **IMPLEMENTADO**

**Requisitos do Cliente:**
- Professor insere: nota disciplina, prova seleção, CR
- Cálculo automático: (disciplina×5 + seleção×3 + CR×2) / 10
- Interface por disciplina

**Funcionalidades Implementadas:**
- ✅ Campos para `notaDisciplina`, `notaSelecao`, `coeficienteRendimento`, e `notaFinal` adicionados à tabela `inscricao`.
- ✅ Endpoint `/api/inscricao/$id/grades` para submeter as notas.
- ✅ O endpoint calcula a `notaFinal` automaticamente e a armazena no banco.
- ✅ Hook `useApplicationGrading` criado em `src/hooks/use-inscricao.ts` para interagir com a API.
- ✅ UI em `/home/professor/grade-applications` onde o professor pode selecionar um projeto e inserir as notas para cada candidato.
- ✅ A UI exibe a nota final calculada após salvar.

#### 3.2 Geração de Atas de Seleção

**Status Atual:** 🚧 **PARCIALMENTE IMPLEMENTADO**

**Requisitos do Cliente:**
- Gerar ata automática da reunião de seleção
- Incluir classificação e notas
- Campos para assinaturas

**Funcionalidades Implementadas:**
- ✅ Tabela `ata_selecao` adicionada ao schema para versionamento e rastreamento.
- ✅ Endpoint `/api/projeto/$id/gerar-ata-data` que coleta e formata os dados necessários para a ata.
- ✅ UI em `/home/professor/gerar-ata` que permite ao professor selecionar um projeto e gerar os dados da ata.
- ❌ **FALTA**: Template de PDF para a ata.
- ❌ **FALTA**: Fluxo completo de assinatura da ata pelo professor.

#### 3.3 Publicação de Resultados

**Status Atual:** 🚧 **PARCIALMENTE IMPLEMENTADO**

**Requisitos do Cliente:**
- Gerar PDF com resultados por disciplina
- Publicar para alunos consultarem
- Notificar aprovados/reprovados

**Funcionalidades Implementadas:**
- ✅ Endpoint `/api/projeto/$id/publish-results-data` que coleta e formata os dados dos aprovados.
- ✅ UI em `/home/professor/publish-results` para visualizar os resultados.
- ❌ **FALTA**: Template de PDF para o resultado final.
- ❌ **FALTA**: Notificação automática para os alunos.

### Módulo 4: Confirmação e Cadastro de Monitores (Alunos, Professores, Admin)

#### 4.1 Coleta de Dados Bancários do Aluno

**Status Atual:** ✅ **IMPLEMENTADO**

**Requisitos do Cliente:**
- Coletar dados bancários de alunos para pagamento de bolsas.

**Funcionalidades Implementadas:**
- ✅ Campos `banco`, `agencia`, `conta`, `digitoConta` adicionados à `alunoTable`.
- ✅ O perfil do aluno (`/home/profile/page.tsx`) foi atualizado com um formulário para coletar e editar essas informações.
- ✅ O endpoint `/api/users/profile` no `userRouter` foi atualizado para salvar e recuperar os dados bancários.

#### 4.2 Fluxo de Aceite com Validações

**Status Atual:** 🚧 **Endpoint existe mas sem validação completa**

**Requisitos do Cliente:**
- Limite de 1 bolsa por aluno/semestre
- Múltiplas vagas voluntárias permitidas
- Prazo para aceite/recusa

**Pendências/Melhorias:**
- [ ] Validação de bolsa única no backend
- [ ] Controle de prazos
- [ ] Interface clara de aceite para o aluno

#### 4.3 Geração de Termos de Compromisso

**Status Atual:** ❌ **NÃO IMPLEMENTADO**

**Requisitos do Cliente:**
- Gerar termo personalizado por aluno
- Incluir dados do projeto e monitor
- Campos para assinaturas

**Pendências/Melhorias:**
- [ ] Template de termo
- [ ] Personalização por tipo de vaga
- [ ] Rastreamento de assinaturas

#### 4.4 Consolidação Final para PROGRAD

**Status Atual:** 🚧 **Endpoint existe mas formato incorreto e dados incompletos**

**Requisitos do Cliente:**
- Planilha de bolsistas com todos os dados (incluindo bancários)
- Planilha de voluntários separada
- Formato específico PROGRAD

**Pendências/Melhorias:**
- [ ] Mapeamento exato dos campos
- [ ] Separação por tipo de monitor
- [ ] Validação de dados completos antes da exportação

## 4. Status Atual e Lacunas Identificadas (Janeiro 2025)

### Análise dos Endpoints Existentes (src/server/api/root.ts)

**Endpoints Implementados:**
- ✅ `me`: Autenticação e perfil do usuário
- ✅ `course`: CRUD de cursos 
- ✅ `discipline`: CRUD de disciplinas
- ✅ `file`: Gerenciamento de arquivos/upload
- ✅ `onboarding`: Processo de cadastro inicial
- ✅ `edital`: Geração e gestão de editais internos
- ✅ `departamento`: CRUD de departamentos
- ✅ `projeto`: Gestão completa de projetos de monitoria
- ✅ `inscricao`: Sistema de inscrições de alunos (agora com suporte a documentos)
- ✅ `signature`: Sistema de assinatura digital
- ✅ `user`: Gestão de usuários (agora com suporte a dados bancários)
- ✅ `importProjects`: Importação de planejamento semestral
- ✅ `scholarshipAllocation`: Alocação de bolsas
- ✅ `inviteProfessor`: Convite de professores
- ✅ `projetoTemplates`: Templates de projetos
- ✅ `relatorios`: Relatórios para PROGRAD
- ✅ `analytics`: Dashboard de métricas
- ✅ `apiKey`: Gestão de chaves API

**Endpoints FALTANDO (Críticos):**
- ❌ `selecao`: Geração de atas, publicação de resultados, classificações
- ❌ `termos`: Geração e assinatura de termos de compromisso
- ❌ `vagas`: Aceite/recusa de vagas com validações
- ❌ `notificacoes`: Sistema de notificações e lembretes

### Páginas Frontend Órfãs (Sem Backend)

**Páginas implementadas mas sem router backend correspondente:**
- `/professor/atas-selecao` → Precisa do `selecaoRouter`
- `/professor/termos-compromisso` → Precisa do `termosRouter`
- `/professor/publicar-resultados` → Precisa do `selecaoRouter`
- `/admin/consolidacao-prograd` → Precisa expandir `relatoriosRouter`

### Status por Módulo (Atualizado)

#### **Módulo 1: Gestão de Projetos** ✅ **100% COMPLETO**

#### **Módulo 2: Editais e Inscrições** ✅ **95% COMPLETO**
- ✅ Gestão de editais (`editalRouter`)
- ✅ Alocação de bolsas (`scholarshipAllocationRouter`)
- ✅ Sistema de inscrições (`inscricaoRouter`)
- ✅ Validação de documentos por inscrição (`inscricaoRouter`)

#### **Módulo 3: Seleção e Atas** 🚧 **60% COMPLETO**
- ✅ Avaliação de candidatos (páginas e endpoints existem)
- ❌ **FALTA**: `selecaoRouter` para atas e resultados
- ❌ **FALTA**: Publicação automática de resultados
- ❌ **FALTA**: Notificação para alunos

#### **Módulo 4: Cadastro Final** 🚧 **50% COMPLETO**
- ✅ Coleta de dados bancários
- ❌ **FALTA**: `termosRouter` para termos de compromisso
- ❌ **FALTA**: `vagasRouter` para aceite com validações
- ❌ **FALTA**: Consolidação final para PROGRAD
- ❌ **FALTA**: Validação de limite de bolsas

## 5. Próximos Passos Prioritários (ATUALIZADOS)

### **FASE 1 - Completar Módulo 3 (URGENTE)**

#### 1.1 Implementar `selecaoRouter`
```typescript
// Criar: src/server/api/routers/selecao/selecao.ts
export const selecaoRouter = createTRPCRouter({
  generateAta: protectedProcedure // Gerar atas de seleção
  publishResults: protectedProcedure // Publicar resultados  
  getApplicationsForGrading: protectedProcedure // Candidatos por projeto
  notifyStudents: protectedProcedure // Notificar alunos sobre resultados
});
```

#### 1.2 Conectar páginas órfãs
- Conectar `/professor/atas-selecao` ao `selecaoRouter.generateAta`
- Conectar `/professor/publicar-resultados` ao `selecaoRouter.publishResults`

### **FASE 2 - Implementar Módulo 4 (CRÍTICO)**

#### 2.1 Implementar `vagasRouter`
```typescript
// Criar: src/server/api/routers/vagas/vagas.ts
export const vagasRouter = createTRPCRouter({
  acceptVaga: protectedProcedure // Aceitar vaga com validação de bolsa única
  rejectVaga: protectedProcedure // Recusar vaga
  getMyVagas: protectedProcedure // Vagas do aluno
  validateBolsaLimit: protectedProcedure // Validar limite de 1 bolsa
});
```

#### 2.2 Implementar `termosRouter`
```typescript
// Criar: src/server/api/routers/termos/termos.ts
export const termosRouter = createTRPCRouter({
  generateTermo: protectedProcedure // Gerar termo de compromisso
  signTermo: protectedProcedure // Assinar termo digitalmente
  getTermosStatus: protectedProcedure // Status de assinaturas
  downloadTermo: protectedProcedure // Download do termo
});
```

#### 2.3 Expandir `relatoriosRouter`
```typescript
// Adicionar ao relatoriosRouter existente:
monitoresFinal: protectedProcedure // Planilha final de monitores
validateCompleteData: protectedProcedure // Validar dados completos
exportConsolidated: protectedProcedure // Exportação consolidada
```

### **FASE 3 - Sistema de Notificações**

#### 3.1 Implementar `notificacoesRouter`
```typescript
// Criar: src/server/api/routers/notificacoes/notificacoes.ts
export const notificacoesRouter = createTRPCRouter({
  sendReminders: protectedProcedure // Lembretes automáticos
  getHistory: protectedProcedure // Histórico de notificações
  markAsRead: protectedProcedure // Marcar como lida
});
```

### **FASE 4 - Atualizar root.ts**

```typescript
// Adicionar ao appRouter em src/server/api/root.ts:
export const appRouter = createTRPCRouter({
  // ... routers existentes
  selecao: selecaoRouter,        // NOVO
  termos: termosRouter,          // NOVO  
  vagas: vagasRouter,            // NOVO
  notificacoes: notificacoesRouter, // NOVO
});
```

## 6. Estimativa de Esforço

**Módulo 3 (selecaoRouter):** 2-3 dias
**Módulo 4 (vagas + termos):** 4-5 dias  
**Sistema de notificações:** 2 dias
**Consolidação PROGRAD:** 1-2 dias
**Testes e ajustes:** 2-3 dias

**TOTAL ESTIMADO:** 11-15 dias para completar 100% dos requisitos

O sistema possui uma arquitetura robusta e já implementou 75% dos requisitos. Os 4 routers faltantes (`selecao`, `termos`, `vagas`, `notificacoes`) são críticos para completar o fluxo de monitoria conforme especificado nas transcrições das reuniões.

### Implementação Imediata Recomendada

1. **`selecaoRouter`** - Para conectar as páginas órfãs de atas e resultados
2. **`vagasRouter`** - Para implementar as validações de aceite de bolsas  
3. **`termosRouter`** - Para geração e assinatura digital de termos
4. **Expansão do `relatoriosRouter`** - Para consolidação final PROGRAD

### Considerações Técnicas

Todas as implementações devem seguir:
- **API and Hooks Pattern** (Cursor Rules existentes)
- **Code Development Guidelines** com foco em TypeScript strict
- Padrão de commits convencionais
- Reutilização dos padrões existentes (assinatura digital, upload de arquivos)
- Integração com sistema de notificações existente

### Contexto Crítico para Implementação

Com base nas transcrições, os pontos mais urgentes são:

1. **Validação de bolsa única** - Aluno só pode ter 1 bolsa por semestre
2. **Termos de compromisso** - Professor e aluno precisam assinar digitalmente
3. **Consolidação PROGRAD** - Planilhas finais separadas (bolsistas vs voluntários)
4. **Notificações automáticas** - Email para alunos sobre resultados

O sistema já possui toda a infraestrutura necessária (assinatura digital, PDFs, email, validações). Os novos routers apenas conectarão as funcionalidades existentes ao fluxo completo de monitoria.