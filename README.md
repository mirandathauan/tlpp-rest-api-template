# tlpp-rest-api-template

Template de API REST em **TLPP** para **TOTVS Protheus**, usando o REST nativo do tlppCore (anotações `@Get` e o objeto `oRest`): autenticação por token, paginação, consultas com bind de parâmetros e respostas JSON com status HTTP corretos.

Use como ponto de partida para suas próprias APIs, em vez de copiar um fonte antigo e apagar metade dele.

## O que tem aqui

| Método | Endpoint | Descrição |
|---|---|---|
| GET | `/api/v1/clientes` | Lista clientes (SA1) com paginação e filtro por nome |
| GET | `/api/v1/clientes/:codigo` | Retorna um cliente específico (`?loja=01` opcional; sem ela, retorna a primeira loja) |

**Recursos:**

- 🔐 Autenticação via header próprio `X-Api-Token`, lido com `oRest:getHeaderRequest()`. Funciona também em servidores com `security=1`, que usam o `Authorization` para o usuário e a senha do Protheus
- 📄 Paginação no padrão `items` + `hasNext` (`?page=1&pageSize=10`, máximo 100)
- 🛡️ Consultas com `FWExecStatement`: valores passados por parâmetro (`?`), com bind feito no banco
- 📦 Status HTTP com `oRest:setStatusCode()` (o padrão do REST tlppCore é 200) e `Content-Type` JSON com `oRest:setKeyHeaderResponse()`
- 🧩 Funções auxiliares reutilizáveis (`responder`, `responderErro`, `paramNumerico`, `paramTexto`)

O exemplo usa a tabela padrão **SA1 (Clientes)**, então funciona em qualquer ambiente Protheus.

## Requisitos

- Protheus com **REST tlppCore** habilitado (`tlpp-core.th` / `tlpp-rest.th`)
- LIB com a classe `FWExecStatement`
- Serviço REST configurado e ativo no `appserver.ini` (consulte o TDN para o seu release)

## Como usar

1. Compile `clientes.api.tlpp` no RPO.
2. No Configurador (SIGACFG), crie o parâmetro **`MV_XAPITOK`** (tipo caractere) com o token da API.
3. Reinicie o serviço REST, se necessário.
4. Teste, ajustando host, porta e prefixo de URL (`url=` da seção do `httpuri`) conforme o seu `appserver.ini`:

```bash
curl -H "X-Api-Token: SEU_TOKEN" \
  "https://localhost:2200/api/api/v1/clientes?page=1&pageSize=5&nome=silva"
```

Se o REST estiver com `security=1`, envie também o usuário e a senha do Protheus:

```bash
curl -u usuario:senha -H "X-Api-Token: SEU_TOKEN" \
  "https://localhost:2200/api/api/v1/clientes"
```

Com certificado autoassinado, adicione `-k` ao `curl`.

Exemplo de resposta:

```json
{
  "items": [
    { "codigo": "000001", "loja": "01", "nome": "JOAO DA SILVA", "municipio": "SALVADOR", "estado": "BA" }
  ],
  "hasNext": false
}
```

Sem token ou com token errado:

```json
{ "code": 401, "message": "Token invalido ou ausente." }
```

## Encoding

O fonte está salvo em **UTF-8** para ser lido corretamente no GitHub. A função `EncodeUTF8` converte de CP1252 (padrão do Protheus e do banco) para UTF-8. Por isso, as mensagens fixas do fonte não usam acentos: escritas em um arquivo UTF-8, elas seriam convertidas duas vezes. Os dados vindos do banco, como o nome do cliente, são convertidos normalmente.

## Pontos de atenção

- **Compare strings com `==`.** Segundo a TDN (Operadores Comuns), com `=` duas strings de tamanhos diferentes podem ser consideradas iguais: `"ABCD" = "ABC"` retorna verdadeiro, enquanto `"ABCD" == "ABC"` retorna falso. Numa validação de token escrita como `cEsperado = cRecebido`, isso significaria aceitar só o começo do token.
- **Não use o `Authorization` para o token da API** se o REST estiver com `security=1`: o servidor usa esse header para o usuário e a senha do Protheus.
- **Execute a query pela própria classe.** O template usa `oExec:OpenAlias()` em vez de pegar a query pronta e abrir com outra função.
- **Token fixo é o mínimo.** Em produção, avalie a autenticação nativa do REST do Protheus ou um token com expiração, sempre sobre **HTTPS**.
- **Paginação com `DbSkip`** é simples e funciona em qualquer banco. Para tabelas grandes, pagine direto no SQL.
- **LGPD**: o exemplo retorna apenas dados cadastrais não sensíveis.

## Referências oficiais (TDN - TOTVS)

- [Operadores Comuns (`=` e `==`)](https://tdn.totvs.com/display/tec/Operadores+Comuns)
- [FWExecStatement](https://tdn.totvs.com/display/public/framework/FWExecStatement)
- [REST tlppCore: Query String / objeto oRest](https://tdn.totvs.com/display/tec/Query+String)
- [oRest:getHeaderRequest](https://tdn.totvs.com/display/tec/oRest:getHeaderRequest)
- [HTTP Status Code (oRest:setStatusCode)](https://tdn.totvs.com/display/tec/HTTP+Status+Code)
- [oRest:setKeyHeaderResponse](https://tdn.totvs.com/display/tec/oRest:setKeyHeaderResponse)
- [Formato da Mensagem (Content-Type)](https://tdn.totvs.com.br/display/tec/Formato+da+Mensagem)

## Contribuindo

Issues e PRs são bem-vindos: novos endpoints de exemplo (POST/PUT), testes, melhorias de segurança.

## Licença

MIT
