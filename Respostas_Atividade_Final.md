# Atividade Final - DevOps (Unidade 3)
**Integrantes:** João Pedro Sousa Martinho e Maria Eduarda Caetana

---

## Parte A - Diagnóstico do Ambiente

Bancos de dados em versões diferentes:

Problema: O ambiente de homologação roda uma versão diferente do banco de produção.

Consequência: Os testes podem dar falso positivo. Uma consulta ou script pode funcionar no teste e falhar na produção quando os usuários reais acessarem.

Estado desejado: Usar exatamente a mesma versão de SGBD nos dois ambientes, controlando os parâmetros por arquivo de configuração.

Evidência: Logs de inicialização ou comandos de versão idênticos em ambos os ambientes.
(Camada afetada: Software de base / Dados | Classificação: Configuração e Implantação)

Porta liberada manualmente (Drift):

Problema: Alguém alterou uma regra de rede ou porta de forma manual direto no ambiente de produção.

Consequência: Gera falhas de segurança e faz o ambiente real ficar diferente do que está documentado no código oficial.

Estado desejado: Toda a infraestrutura gerenciada estritamente via código (IaC), sem alterações manuais.

Evidência: Rodar o comando terraform plan e o terminal retornar No changes.
(Camada afetada: Configurações / Rede | Classificação: Gerenciamento)

Configuração dependente de uma única pessoa:

Problema: O conhecimento de como montar e ajustar o ambiente está apenas na cabeça de um colaborador que pode se ausentar.

Consequência: Se a pessoa faltar ou sair da empresa, o time fica travado para subir o sistema ou resolver um problema crítico.

Estado desejado: Processos automatizados e descritos em arquivos versionados no repositório.

Evidência: Histórico de commits no GitHub com os scripts de automação acessíveis para toda a equipe.
(Camada afetada: Configurações / Gerenciamento | Classificação: Gerenciamento e Provisionamento)

---

## Parte B - Proposta de Mudança e Revisão

O que foi alterado no código: No laboratório de Terraform (terraform/main.tf), a linha de conteúdo do arquivo foi alterada de content = "AMBIENTE=homologacao\n" para incluir a nova linha: content = "AMBIENTE=homologacao\nVERSAO=2\n".

Qual efeito era esperado: Esperava-se que o Terraform detectasse a divergência entre a configuração desejada e o estado atual, gerando um plano para substituir o arquivo local e atualizar o seu conteúdo.

Como a alteração foi executada: A alteração foi salva no editor. Em seguida, gerou-se o plano com terraform plan -out=plano-mudanca (que indicou 1 to add, 1 to destroy) e a mudança foi aplicada executando terraform apply plano-mudanca.

Como o resultado foi verificado: Utilizou-se o comando cat terraform/ambiente.txt via terminal para inspecionar o conteúdo e comprovar que as duas linhas (AMBIENTE=homologacao e VERSAO=2) foram gravadas com sucesso.

O que deveria permanecer sem alteração: O nome do arquivo (ambiente.txt), o diretório de destino e as permissões de acesso originais (0644).

Como ocorreu a revisão pelos dois integrantes: O [Integrante A] criou uma branch dedicada, realizou a modificação no código do Terraform e abriu a proposta. O [Integrante B] revisou o plano de execução gerado, validou que a destruição controlada do arquivo não geraria perda de dados e aprovou a mesclagem para o fluxo principal.

---

## Parte C - Falha e Recuperação

Sintoma observado e hipótese: Durante a execução do playbook do Ansible, o terminal retornou um erro em vermelho (failed=1). A hipótese imediata foi que o módulo copy tentou criar um arquivo dentro de uma pasta que simplesmente não existia na árvore de diretórios do projeto.

Linha de configuração envolvida: A linha com o erro no arquivo ansible/ambiente.yml era: dest: "{{ playbook_dir }}/pasta-inexistente/ambiente.txt".

Correção realizada: Alteramos o caminho de volta para o diretório correto: dest: "{{ playbook_dir }}/ambiente.txt".

Verificação do resultado: Rodamos o playbook de novo. O terminal mostrou failed=0 e changed=0 (provando a idempotência, já que o arquivo correto já estava ajustado), e o comando cat ansible/ambiente.txt confirmou que o conteúdo permaneceu íntegro e correto.

---
