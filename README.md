# SIGCH, documentação técnica

SIGCH (Sistema Integrado de Gestão de Chamados) é uma aplicação web para abrir e
acompanhar chamados de manutenção e suporte. Está em produção no Hospital
Municipal de Pacatuba, no Ceará, desde julho de 2026.

O código é privado porque o sistema roda em ambiente hospitalar com dados reais.
Este repositório explica o que o sistema faz e por que foi construído assim.

## O problema

Os setores do hospital precisavam de um lugar único para pedir manutenção e
suporte. O SIGCH dá a cada pedido prioridade, responsável e registro do tempo de
atendimento, e entrega à gestão os números do período.

## Resultados

- 74 chamados registrados, 91% concluídos.
- Tempo médio de resolução de 8h11min, calculado pelo próprio banco.
- Inscrito no Prêmio de Inovação ISV 2026, do Instituto São Vicente, no eixo
  Eficiência Operacional e Processos.

## Stack

React 19, React Router 7, Vite, Tailwind CSS 4, Supabase (Auth, PostgreSQL com
RLS, Storage, Realtime e Edge Functions), Recharts, jsPDF e deploy na Vercel.

## Modelo de dados

O sistema é multi-estabelecimento. Cada setor pertence a um estabelecimento, e
cada usuário, menos o desenvolvedor, pertence a exatamente um.

| Entidade | Papel |
|---|---|
| Estabelecimentos | Unidades atendidas: hospital, clínica ou outra organização |
| Setores | Onde o problema acontece. Desativar um setor tira ele da abertura de chamados, mas preserva o histórico |
| Chamados | Título, descrição, setor, prioridade, status, solicitante, responsável e datas de abertura, início e conclusão |
| Categorias de atendimento | O perfil profissional que atende, como eletricista ou suporte de TI |
| Histórico | Cada mudança de status, com usuário, valor anterior, valor novo e IP de origem |
| Comentários e anexos | Conversa entre solicitante e técnico, com fotos |
| Notificações | Avisos de mudança de status e de comentário novo, entregues em tempo real |

Status de um chamado: aberto, em atendimento, aguardando peça, aguardando setor,
concluído e cancelado. Prioridades: baixa, média, alta e crítica.

## Perfis e permissões

| Perfil | O que pode |
|---|---|
| Solicitante | Abre chamados e acompanha só os próprios |
| Técnico | Vê os chamados atribuídos a ele e os sem responsável das categorias liberadas para ele |
| Administrador | Acessa tudo do próprio estabelecimento: chamados, usuários, setores, relatórios e painéis |
| Desenvolvedor | Gerencia estabelecimentos e entra em qualquer um para dar suporte |

## Decisões

**A autorização fica no banco, não na tela.** Toda regra de acesso é uma policy
de Row Level Security. Chamar a API do Supabase direto, com a chave pública, não
contorna nenhuma delas.

**O cadastro não escolhe o próprio perfil.** Toda conta nova nasce como
solicitante. Um trigger ignora o perfil enviado pelo navegador.

**O solicitante não altera o chamado depois de aberto.** A RLS do Postgres
restringe por linha, não por coluna. Se o solicitante pudesse atualizar o
próprio chamado, também poderia concluí-lo sozinho ou escolher o responsável.
Por isso o update ficou só com quem atende.

**O tempo de resolução é calculado por trigger.** Quando o chamado é concluído,
o banco calcula a duração. O número dos relatórios não depende do relógio do
navegador.

**O histórico não pode ser editado.** Update e delete estão revogados na tabela
de histórico, que funciona como trilha de auditoria.

**Dados pessoais dos colegas não ficam visíveis para todos.** Quem não administra
lê uma view com apenas id, nome e setor de cada pessoa.

**O painel de TV não toca na tabela de chamados.** Ele roda sem login, pelo
código do painel, e recebe só os campos necessários: prioridade, status, setor e
responsável. Atualiza a cada 15 segundos.

**As fotos ficam em bucket privado.** A leitura passa por URL assinada de curta
duração. Antes do upload, a imagem é redesenhada em canvas, o que remove os
metadados EXIF e GPS e descarta qualquer conteúdo que não seja imagem.

**Conta compartilhada exige plantão.** Setores com escala, como o Raio-X, usam um
login só. Quem usa a conta escolhe o profissional do turno, e o banco recusa a
abertura de chamado sem plantão ativo.

## Segurança

Uma revisão com base no OWASP encontrou falhas de controle de acesso
introduzidas na migração de um estabelecimento para vários: funções que
ignoravam a RLS sem checar o estabelecimento e um cadastro que aceitava dados de
outro cliente. As correções foram aplicadas e a regra passou a ser revisar toda
função privilegiada sempre que o modelo de dados ganha um conceito novo de
isolamento.

Também foi feito um levantamento de conformidade com a LGPD, com documentação
dos dados pessoais guardados e de quem pode acessá-los.

## Autor

Gabriel Pedrosa. [Portfólio](https://gabrielpedrosa0.github.io) ·
[LinkedIn](https://www.linkedin.com/in/gabriel-pedrosa-6618a6343/)
