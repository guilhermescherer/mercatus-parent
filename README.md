# mercatus-parent

Parent POM Maven compartilhado pelos serviços do Mercatus. Centraliza versões,
dependências gerenciadas e configuração de plugins. Cada serviço continua sendo
um repositório e um build independente; este POM não agrega módulos.

## Configuração atual

| Item | Valor |
|---|---|
| Coordenadas | `com.mercatus:mercatus-parent:1.0.0` |
| Packaging | `pom` |
| Java | `25` |
| Spring Boot parent | `4.1.0` |
| Testcontainers BOM | `2.0.5` |
| ArchUnit | `1.5.1`, integração `archunit-junit6` |
| JUnit Jupiter | `6.0.3`, gerenciado pelo Spring Boot |

O Surefire exclui `*IT.java`; o Failsafe inclui esses arquivos e vincula
`integration-test` e `verify`. A configuração fica em `pluginManagement`:
o serviço declara o Failsafe em `build/plugins` para ativá-lo, como fazem os
serviços existentes. O plugin Maven do Flyway também fica em `pluginManagement`;
a execução das migrations no start depende das dependências e da configuração
Flyway de cada serviço.

## Instalação local

Pré-requisitos: JDK 25 e Maven 3.9+. Confirme o Java efetivamente usado pelo Maven:

```bash
mvn -version
```

Se necessário, ajuste `JAVA_HOME` para seu JDK 25 antes de executar os comandos.
Na pasta deste repositório:

```bash
mvn verify
mvn install
```

`install` disponibiliza o POM no repositório Maven local. Isso permite compilar
os serviços sem buscar o parent no GitHub Packages. O parent não contém código
de aplicação nem testes próprios; os testes são executados nos serviços.

## Consumo pelos serviços

Cada serviço declara:

```xml
<parent>
    <groupId>com.mercatus</groupId>
    <artifactId>mercatus-parent</artifactId>
    <version>1.0.0</version>
    <relativePath/>
</parent>
```

Com `relativePath` vazio, o Maven resolve o artefato pelo repositório local ou
por um repositório remoto configurado, sem procurar o POM de uma pasta vizinha.
Após mudanças locais neste parent, execute `mvn install` antes de validar os
serviços que devem consumi-las.

## GitHub Packages

O destino de publicação configurado em `distributionManagement` é:

```text
https://maven.pkg.github.com/guilhermescherer/mercatus-parent
```

Para consumir a versão remota, o Maven precisa de um repositório com ID `github`
e credenciais para esse mesmo ID. Catalog e Customer possuem um
`.mvn/settings.xml` versionado que lê `MAVEN_USERNAME` e `MAVEN_PASSWORD`
(usuário GitHub e token com acesso de leitura ao pacote). Esse arquivo precisa
ser passado explicitamente com `-s`; apenas definir as variáveis não o ativa.

Por exemplo, dentro do Catalog ou do Customer, com as credenciais configuradas:

```bash
mvn -s .mvn/settings.xml verify
```

O Inventory pode usar a instalação local ou, na pasta guarda-chuva, o mesmo
settings existente no Catalog. `distributionManagement` define onde publicar;
ele não configura sozinho o repositório de download dos serviços.

## Publicação

O workflow `.github/workflows/publish.yml` executa `mvn -B deploy` a cada push de
tag. Usa JDK 25, servidor Maven `github` e `GITHUB_TOKEN` com `packages: write`.
A versão publicada é a declarada no POM; a tag não altera esse valor.
