# propriedades-e-ambientes-mulesoft
## Para entender as propriedades e variáveis de ambiente no Mulesoft

---

## Passo a Passo: Configurando Propriedades por Ambiente com Arquivos YAML no MuleSoft

1. Estrutura dos Arquivos
- Criar os arquivos `config` dentro de src/main/resources (exemplos):
	- dev.config.yaml
	- hml.config.yaml
	- prod.config.yaml

---

2. Conteúdo dos Arquivos YAML
Cada arquivo terá sua própria configuração. Exemplo para `dev.yaml`:
```yaml
db:
  host: localhost
  port: 3306
  user: devuser
  password: devpass

api:
  key: abc123
```
E o `prod.yaml`, por exemplo:
```yaml
db:
  host: prod.db.server
  port: 3306
  user: produser
  password: supersecurepass

api:
  key: prod_api_key_987654
```

---

3. Configuração do `global.xml` com `Configuration Properties`
No `global.xml`, configure o carregamento do arquivo apropriado:
```xml
<configuration-properties
    doc:name="Propriedades por ambiente"
    file="config/${env}.yaml"
    format="yaml" />
```
*Dica:* Aqui usamos `${env}` como variável para determinar o ambiente ativo.

---

4. Definir o Ambiente com Variável do Sistema
Antes de iniciar a aplicação, você precisa definir a variável de ambiente para que o Mule saiba qual arquivo carregar.
- No Windows:
```bash
set env=dev
```
- No macOS/Linux:
```bash
export env=dev
```
*Observação:* Substituir "dev" por "hml" ou "prod", conforme necessário.

---

5. Utilizar as Propriedades no Código Mule
Dentro de seus flows, você pode acessar as propriedades via:
```xml
<db:select config-ref="DB_Config" doc:name="Consulta">
    <db:sql>SELECT * FROM clientes WHERE cidade = '${db.host}'</db:sql>
</db:select>
```
Ou no DataWeave:
```dw
%dw 2.0
output application/json
---
{
  banco: p("db.host"),
  usuario: p("db.user"),
  chave_api: p("api.key")
}
```

---

6. Configurar os ambientes
- Em `src/main/mule`, onde se encontra o arquivo '.xml' principal (main):
	- Criar um `Listener`
		- Clicar em `Add` e depois em `OK`, para confirmar as configurações para endpoint local, mais para entender o funcionamento.
		- Em General, na opção Path, colocar somente uma "/". Se quiser colocar um nome para o endpoint, pode colocar, também a seu critério.
	- Criar um `Request`
		- Clicar em `Add` e ajustar o protocolo, mudando para 'HTTPS'.
		- Em Host, preencher: `${webservice.url}` e clicar em `OK`. São os dados da variável, configurados em cada um dos arquivos '.yaml', que precisam estar iguais, para cada um dos ambientes. Se criar um dos arquivos, com nome ou extensão diferente, retorna erro, porque não vai reconhecer.
		- Ainda em Request, mantém o método como GET. Em Path, colocar: `/${webservice.env}`, para localizar da variável.

---

7. Fazer os testes locais
Para verificar se está funcionando em cada ambiente desejado, utilizando no Insomnia ou Postman:
- Criar uma request do tipo GET, colocando o endereço: `localhost:8081' (somente, porque não coloquei um nome para o endpoint, declarei no código somente como "/")
Ao rodar o teste, vai identificar o endpoint declarado, conforme o ambiente selecionado, configurado em `Global Configuration Elements`.

---

8. Fazer o deploy no Cloud Hub da Mulesoft (https://anypoint.mulesoft.com/cloudhub)

*Observação*: Se não tiver uma conta, criar em: (https://anypoint.mulesoft.com/login/signin?).

No primeiro teste que fizer, seguindo os parâmetros que foram configurados no teste local, mas diretamente na console, consegue identificar e testar, obtendo o mesmo resultado.

---