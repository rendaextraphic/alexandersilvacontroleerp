# Meu Controle

Dashboard pessoal de **produtividade no trabalho** e **controle financeiro**: salário, produtividade (conferência e separação), despesas, dívidas, livro-caixa, planejamento mensal e **relatórios em PDF A4**.

O app roda no navegador; login e dados ficam em uma conta na nuvem (Supabase). Interface em português do Brasil.

## Recursos

| Módulo | O que faz |
| --- | --- |
| **Início** | Cards, gráficos (receitas × despesas, saldo, categorias, produtividade diária), contas a vencer, dívidas próximas, últimas movimentações, % do salário comprometido, filtros por mês, ano e período personalizado. |
| **Produtividade** | Registros de conferência/separação (vários por dia), histórico por dia/semana/mês, média, melhor dia, comparação com o período anterior, pagamento integral/parcial/pendente, exportação CSV e PDF. |
| **Salário** | Detalhamento transparente (base, adicional noturno por horas realizadas, assiduidade, produtividade, extras, descontos), parâmetros por mês, controle de pagamentos e diferença entre estimado e recebido. |
| **Despesas** | Única, semanal, mensal, anual e parcelada; recorrências geradas sem duplicar; pagar/desfazer; filtros; planejado × realizado. |
| **Dívidas** | Saldo, parcelas, % quitado, pagamentos parciais/totais, previsão de quitação e simulador de pagamentos extras. |
| **Livro-caixa** | Entradas e saídas, saldo realizado × previsto, contas, transferências, lançamentos vinculados a salário/despesa/dívida sem contagem dupla. |
| **Planejamento** | Compromissos do mês, saldo previsto, meta de economia, orçamento por categoria, alertas, metas financeiras. |
| **Relatórios** | 8 relatórios em PDF A4 com pré-visualização, download e impressão. |
| **Configurações** | Todos os valores editáveis, categorias, contas, orçamentos, aparência, backup JSON, CSV, restaurar e apagar dados. |

## Requisitos

- [Node.js](https://nodejs.org) 18 ou superior

## Instalar e executar

```bash
npm install
npm run dev        # abre em http://localhost:5173
```

Outros comandos:

```bash
npm test           # testes dos cálculos e dos relatórios
npm run typecheck  # verificação de tipos
npm run build      # gera a pasta dist/ para publicar
npm run preview    # serve a pasta dist/ localmente
```

## Publicar no GitHub Pages

1. Crie um repositório no GitHub e envie o projeto:

   ```bash
   git init
   git add .
   git commit -m "Meu Controle"
   git branch -M main
   git remote add origin https://github.com/SEU-USUARIO/meu-controle.git
   git push -u origin main
   ```

2. No GitHub, abra **Settings → Pages** e em **Source** escolha **GitHub Actions**.
3. O workflow `.github/workflows/deploy.yml` roda os testes, gera o build e publica a cada push na branch `main`.
4. O site ficará em `https://SEU-USUARIO.github.io/meu-controle/`.

O Vite usa `base: './'` e a navegação usa `#/rota`, por isso o projeto funciona em qualquer subcaminho de hospedagem estática (Netlify, Vercel, Cloudflare Pages, servidor próprio etc.), sem configuração extra.

## Contas na nuvem (login por e-mail e senha)

O app usa o **Supabase** (plano gratuito) para criar contas e guardar os dados. Cada pessoa entra com **e-mail e senha** e vê os próprios dados em **qualquer aparelho ou navegador**. Ninguém consegue ver os dados de outra conta: o banco tem regras (RLS) que só liberam a linha do próprio usuário.

### Configurar (uma vez, uns 10 minutos)

1. Crie uma conta em [supabase.com](https://supabase.com) e um **New project** (anote a senha do banco; não será usada no app).
2. Abra **SQL Editor → New query**, cole o conteúdo de `supabase/schema.sql` e clique em **Run**. Isso cria a tabela e as regras de segurança.
3. Em **Project Settings → API**, copie a **Project URL** e a chave **anon public**.
4. Em **Authentication → Providers → Email**, confira que "Email" está ligado. Para entrar logo depois de criar a conta, desligue **Confirm email**; se deixar ligado, cada pessoa precisa clicar no link que chega por e-mail.
5. Em **Authentication → URL Configuration**, coloque o endereço do seu site (ex.: `https://seu-site.vercel.app`) em **Site URL** e em **Redirect URLs**. Isso faz funcionar o link de "esqueci minha senha".
6. **Local:** copie `.env.example` para `.env`, cole a URL e a chave e rode `npm run dev`.
7. **Vercel:** em **Settings → Environment Variables**, crie `VITE_SUPABASE_URL` e `VITE_SUPABASE_ANON_KEY` (marque Production, Preview e Development) e faça um **Redeploy**. **GitHub Pages:** crie os mesmos dois itens em **Settings → Secrets and variables → Actions → Variables** e use-os no passo de build do workflow (`env:`).

A chave **anon public** pode aparecer no site; ela é pública por desenho e só funciona dentro das regras de segurança do banco. **Nunca** use a chave `service_role` neste projeto.

### Como funciona

- Os dados de cada conta ficam em uma linha da tabela `user_data`, gravada automaticamente a cada alteração.
- Ao voltar para a aba (e a cada minuto) o app busca alterações feitas em outro aparelho.
- Se dois aparelhos editarem ao mesmo tempo, **vale o que foi salvo primeiro**: o outro recebe os dados atualizados e um aviso. Nada é sobrescrito sem aviso.
- Na primeira entrada com a nuvem vazia, dados antigos que estavam neste navegador (versão anterior) são enviados para a conta e a cópia local é apagada.
- Em **Configurações → Dados e backup → Minha conta** é possível trocar a senha e sair.
- Os dados ficam no banco do seu projeto Supabase, protegidos por login e RLS (não têm criptografia ponta a ponta; quem administra o projeto Supabase consegue ver o banco). O **backup JSON/CSV exportado não tem senha**.

## Como os cálculos funcionam

- **Produção** = quantidade × valor unitário. Valores monetários são arredondados a 2 casas em um único ponto (`round2`).
- **Total bruto** = salário-base + adicional noturno + assiduidade + produtividade elegível + horas extras + outros adicionais.
- **Líquido estimado** = bruto − descontos (vale-transporte, INSS informado, adiantamentos, outros).
- **Adicional noturno** = valor da hora normal × % × horas noturnas realizadas (horas/dia × dias trabalhados, ou o valor informado para o mês). Não é aplicado sobre o salário inteiro. Há opção de hora noturna reduzida.
- **INSS e descontos legais** não são calculados com alíquotas embutidas: você informa valor ou percentual conforme o holerite; sem isso, aparece como "a confirmar".
- **Estimativa × folha confirmada**: ao registrar o pagamento como recebido/ajustado, ele passa a valer no lugar da estimativa e a diferença é exibida.
- **Sem contagem dupla**: produtividade elegível já está no salário (não é receita extra); pagamento de dívida é uma única saída (não é também despesa); transferências não são receita nem despesa; lançamentos "já no salário" não entram nos saldos.
- **Juros e multas** só entram nas simulações quando você informa a taxa.

Os cálculos ficam em `src/lib/calc.ts` e são cobertos por testes (`src/lib/calc.test.ts`).

## Estrutura

```
src/
  lib/          cálculos, datas, formatação, validação de backup, CSV
  auth/         tela de login e cliente da nuvem (Supabase)
  store/        estado global + gravação/sincronização na nuvem
  components/   layout, UI reutilizável, filtros, gráficos
  pages/        Início, Produtividade, Salário, Despesas, Dívidas, Livro-caixa, Planejamento, Relatórios, Configurações
  reports/      geração dos PDFs A4 (jsPDF + AutoTable)
```

## Limitações conhecidas

- Precisa de conexão com a internet para entrar e salvar; sem internet, as alterações ficam na tela e são reenviadas quando a conexão voltar (se a aba continuar aberta).
- Os valores de salário são **estimativas**; confira sempre com o holerite. A regra do adicional noturno e a jornada precisam refletir o seu contrato.
- Os PDFs usam a fonte padrão (Helvetica) e portanto caracteres fora do alfabeto latino não são impressos.
- Alguns navegadores de celular não exibem PDF dentro da página; use "Abrir em nova aba" ou "Baixar PDF".
- A simulação de quitação é aproximada (juros mensais compostos sobre o saldo) e não substitui o cálculo do credor.
- Despesas recorrentes são geradas para uma janela à frente (12 meses para mensais) e completadas automaticamente.
