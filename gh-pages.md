Uma prática muito importante no desenvolvimento de software em equipe é o Code Review, usado para garantir a qualidade do código, ajudar a disseminar conhecimento entre os membros do time e evitar que erros que não foram pegos pelo processo automatizado passem para as branchs estáveis.

O processo de Code Review leva tempo e exige dedicação dos envolvidos, mas apesar de ser muito útil, é comum que as pessoas se esqueçam ou às vezes nem pensem em checar detalhes que deveriam ser corrigidos durante o Code Review.

Felizmente, isso não é mais um problema. Os criadores do Cocoapods e Fastlane, ferramentas já bem conhecidas no mundo do desenvolvimento iOS, criaram a Danger.

Danger é uma ferramenta escrita em Ruby que veio para auxiliar o processo de Code Review para todos os desenvolvedores, independente da plataforma. Ela garante que tudo que foi definido que seria analisado durante o Code Review seja de fato analisado. E, a melhor parte, de forma automatizada, integrada com o servidor de integração contínua e a plataforma de controle de versão.

Durante o processo de build do projeto a Danger vai analisar as regras definidas em um script Ruby e permitir que o time codifique suas normas de Code Review, deixando que as pessoas usem seu tempo para pensar em tarefas mais complicadas. De acordo com o que foi definido, a Danger vai comentar nos pull requests o que está de acordo ou não, permitindo também quebrar o processo de build caso alguma regra mais importante seja violada. Conforme as correções vão sendo feitas as mensagens são alteradas para deixar a evolução que ocorreu transparente .

Implementar a Danger em seu projeto é bem simples, basta cinco passos:

- Instale a gem 'danger' no projeto. É recomendável que isso seja feito utilizando o Bundler, um gerenciador de dependências Ruby. Com a gem instalada você pode utilizar o comando `Danger init`, que irá guiá-lo pelos próximos passos.

- Crie um arquivo vazio chamado Dangerfile. É neste arquivo que será escrito o script em Ruby que ditará as regras seguidas para o seu Code Review. Caso você não esteja familiarizado com Ruby não se preocupe, apesar de ser a linguagem utilizada não é necessário conhecimento prévio, basta conhecer conceitos básicos de programação. Além disso, a escrita do script é feita de forma quase natural.

- Este passo não é necessário para utilizar a Danger, mas sim para que ela possa comentar diretamente nos Pull Requests. Crie uma conta em sua plataforma de controle de versão (Github, Gitlab, Bitbucket Server). Esta conta será usada como um bot pela Danger para comentar nos Pull Requests.
A conta é criada como uma conta de usuário normal e a configuração para ser usada como bot vária de uma plataforma para outra. Sugiro que leia a documentação sobre o assunto na plataforma que for usar.

- Adicione ao seu servidor de integração contínua uma variável de ambiente chamada `DANGER_GITHUB_API_TOKEN`. O valor desta variável é fornecido pela plataforma de controle de versão e varia de uma para a outra, mas pode ser facilmente achada utilizando a documentação da plataforma.

- Adicione ao processo de build de seu servidor de integração contínua a Danger. Novamente, a forma como isto deve ser feito pode variar de um servidor para o outro, mas o que deve ser feito é adicionar o comando `danger` (ou `bundle exec danger`) à sua pipeline e pronto, já está tudo preparado.

# Danger em uso

Aqui temos um exemplo de uma Dangerfile, agora vamos explicar o que cada parte do script está fazendo.

![screenshots/Dangerfile.png](https://github.com/brurend/DangerPoc/tree/gh-pages/screenshots/Dangerfile.png)

```Ruby
# Warn when there is a big PR
warn("Big PR") if git.lines_of_code > 5000
```
Um aviso é dado se o Pull Request for muito grande e estiver com mais linhas do que o definido.

```Ruby
# Force tests to be made within all PRs that changes files
has_app_changes = !git.modified_files.grep(/DangerPoc/).empty?
has_test_changes = !git.modified_files.grep(/DangerPocTests/).empty?

if has_app_changes && !has_test_changes
  fail "Tests were not updated"
end
```
Define duas variáveis locais e checa se tiveram arquivos modificados mas não foram adicionados testes destes arquivos. Neste caso a build irá falhar, pois viola uma regra definida como primordial por este time.

![screenshots/PR_Tests.png](https://github.com/brurend/DangerPoc/tree/gh-pages/screenshots/PR_Tests.png)

```Ruby
# Make a note about contributors not in the organization
unless github.api.organization_member?('DangerPoc', github.pr_author)
  message "@#{github.pr_author} is not a contributor yet"
end
```
Aparece uma mensagem se o autor do Pull Request não for um colaborador do projeto.

![screenshots/Contributor.png](https://github.com/brurend/DangerPoc/tree/gh-pages/screenshots/Contributor.png)

```Ruby
# Fail the build based on code coverage
xcov.report(
  workspace: "DangerPoc.xcworkspace",
  scheme: "DangerPoc",
  minimum_coverage_percentage: 50.0
)
```
Aqui está sendo utilizado um plugin para a Danger. Xcov é uma ferramenta usada para geração de relatório de cobertura de testes em iOS.
Este plugin permite que a Danger falhe a build caso a cobertura de código esteja abaixo do esperado, que neste caso é de 50%.

![screenshots/Contributor_CodeCoverageUnder.png](https://github.com/brurend/DangerPoc/tree/gh-pages/screenshots/Contributor_CodeCoverageUnder.png)
![screenshots/CodeCoverageOk.png](https://github.com/brurend/DangerPoc/tree/gh-pages/screenshots/CodeCoverageOk.png)
![screenshots/Xcov.png](https://github.com/brurend/DangerPoc/tree/gh-pages/screenshots/Xcov.png)

```Ruby
commit_lint.check warn: :all
```
Outro plugin muito útil para garantir que as mensagens de commit estejam sendo escritas de forma clara e respeitando os guidelines conhecidos de git.

![screenshots/CommitLintWarning.png](https://github.com/brurend/DangerPoc/tree/gh-pages/screenshots/CommitLintWarning.png)
![screenshots/CommitLintOk.png](https://github.com/brurend/DangerPoc/tree/gh-pages/screenshots/CommitLintOk.png)

# Conclusão

Danger ainda é uma ferramenta nova, mas está em crescimento e com um potencial enorme. Code Review é algo presente diariamente na vida de quase todos os desenvolvedores e uma ferramenta que possa facilitar e melhorar este processo de forma simples tem tudo para ser incorporada por todos nós em nossos processos de build.

Obs: Danger é referida como "a/ela" apesar de ser uma ferramenta, pois é uma homenagem dos criadores para Gem "Danger" McShane, que esteve envolvida com o processo de concepção e com a idealização do projeto em si.

### Referências:

http://danger.systems

https://github.com/danger/danger
