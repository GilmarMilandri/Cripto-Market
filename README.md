
# Cripto-Market

# React + TypeScript + Vite

# Documentação do Código - Componente Home

## Visão Geral
O componente `Home` é responsável por exibir uma lista de criptomoedas com informações relevantes, como preço, capitalização de mercado, volume de negociação e variação em 24 horas. Além disso, permite que o usuário pesquise uma moeda específica e acesse uma página de detalhes.

## Dependências
Este componente faz uso de:
- `useState`, `useEffect`, e `FormEvent` do React para gerenciamento de estado e manipulação de eventos.
- `react-icons/bs` para exibição do ícone de busca.
- `react-router-dom` para navegação entre páginas.
- `fetch` para buscar dados da API CoinCap.

## Interface `CoinProps`
Define a estrutura dos dados de uma criptomoeda:
```typescript
export interface CoinProps{
    id: string;
    name: string;
    symbol: string;
    priceUsd: string;
    changePercent24Hr: string;
    marketCapUsd: string;
    volumeUsd24Hr: string;
    rank: string;
    supply: string;
    maxSupply: string;
    vwap24Hr: string;
    explorer: string;
    formatedPrice?: string;
    formatedMarket?: string;
    formatedVolume?: string;
}
```

## Estado do Componente
O componente `Home` utiliza os seguintes estados:
- `input`: Armazena o valor digitado na busca.
- `coins`: Lista de criptomoedas carregadas da API.
- `offset`: Controla a paginação dos dados ao buscar mais moedas.

## Ciclo de Vida
A função `getData` é chamada sempre que `offset` é alterado, buscando mais moedas da API.
```typescript
useEffect(() => {
    getData();
}, [offset])
```

## Funções Principais
### `getData()`
Busca dados da API CoinCap e formata os valores de preço, capitalização de mercado e volume:
```typescript
async function getData() {
    fetch(`https://api.coincap.io/v2/assets?limit=10&offset=${offset}`)
    .then(response => response.json())
    .then((data: DataProp) => {
        const coinsData = data.data;

        const price = Intl.NumberFormat("en-US", {
            style: "currency",
            currency: "USD"
        })

        const priceCompact = Intl.NumberFormat("en-US", {
            style: "currency",
            currency: "USD",
            notation: "compact"
        })

        const formatedResult = coinsData.map((item) => {
            return {
                ...item,
                formatedPrice: price.format(Number(item.priceUsd)),
                formatedMarket: priceCompact.format(Number(item.marketCapUsd)),
                formatedVolume: priceCompact.format(Number(item.volumeUsd24Hr)),
            }
        })

        setCoins([...coins, ...formatedResult]);
    })
}
```

### `handleSubmit(e:FormEvent)`
Impede o recarregamento da página ao submeter o formulário e redireciona para a página de detalhes:
```typescript
function handleSubmit(e: FormEvent) {
    e.preventDefault();
    if(input === "") return;
    navigate(`/detail/${input}`)
}
```

### `handleGetMore()`
Atualiza o estado `offset` para carregar mais criptomoedas:
```typescript
function handleGetMore(){
   if(offset === 0){
        setOffset(10);
        return;
    }
    setOffset(offset + 10);
}
```

## Renderização do Componente
### Formulário de Busca
```tsx
<form className={styles.form} onSubmit={handleSubmit}>
    <input
        type="text"
        placeholder="Digite o nome da moeda... EX bitcoin"
        value={input}
        onChange={ (e) => setInput(e.target.value)}
    />
    <button type="submit">
        <BsSearch size={30} color="#FFF"/>
    </button>
</form>
```

### Tabela de Moedas
```tsx
<table>
    <thead>
        <tr>
            <th scope="col">Moeda</th>
            <th scope="col">Valor mercado</th>
            <th scope="col">Preço</th>
            <th scope="col">Volume</th>
            <th scope="col">Mudança 24h</th>
        </tr>
    </thead>
    <tbody id="tbody">
        {coins.length > 0 && coins.map((item) => (
            <tr className={styles.tr} key={item.id}>
                <td className={styles.tdLabel} data-label="Moeda">
                    <div className={styles.name}>
                        <img
                            className={styles.imgLogo}
                            src={`https://assets.coincap.io/assets/icons/${item.symbol.toLowerCase()}@2x.png`}
                            alt="Logo Cripto"
                        />
                        <Link to={`/detail/${item.id}`}>
                            <span>{item.name}</span> | {item.symbol}
                        </Link>
                    </div>
                </td>
                <td className={styles.tdLabel} data-label="Valor mercado">{item.formatedMarket}</td>
                <td className={styles.tdLabel} data-label="Preço">{item.formatedPrice}</td>
                <td className={styles.tdLabel} data-label="Volume">{item.formatedVolume}</td>
                <td className={Number(item.changePercent24Hr) > 0 ? styles.tdProfit : styles.tdLoss} data-label="Mudança 24h">
                    <span>{Number(item.changePercent24Hr).toFixed(2)}%</span>
                </td>
            </tr>
        ))}
    </tbody>
</table>
```

### Botão "Carregar Mais"
```tsx
<button className={styles.buttonMore} onClick={handleGetMore}>
    Carregar mais...
</button>
```

## Conclusão
O componente `Home` consome dados da API CoinCap, exibe uma tabela formatada de criptomoedas e permite busca por moedas específicas. Ele gerencia eficientemente a paginação e atualiza a interface conforme novos dados são carregados.

# Documentação do Código - Componente Detail

## Visão Geral
O componente `Detail` é responsável por exibir informações detalhadas sobre uma criptomoeda específica. Ele obtém os dados da API CoinCap com base no parâmetro da URL, formata os valores e os apresenta de maneira organizada para o usuário.

## Dependências
Este componente utiliza:
- `useState`, `useEffect` do React para gerenciar estado e efeitos colaterais.
- `useParams`, `useNavigate` do `react-router-dom` para obter parâmetros da URL e realizar navegação programática.
- `CoinProps` importado do componente `Home` para manter a tipagem consistente.
- `fetch` para realizar requisições HTTP e obter os dados da API CoinCap.

## Interfaces e Tipos
### `ResponseData`
Representa uma resposta bem-sucedida da API:
```typescript
interface ResponseData{
    data: CoinProps
}
```

### `ErrorData`
Representa uma resposta de erro da API:
```typescript
interface ErrorData{
    error: string
}
```

### `DataProps`
Um tipo que pode ser tanto `ResponseData` quanto `ErrorData`:
```typescript
type DataProps = ResponseData | ErrorData
```

## Estado do Componente
O componente gerencia os seguintes estados:
- `coin`: Armazena os dados da criptomoeda buscada.
- `loading`: Controla o estado de carregamento enquanto os dados estão sendo obtidos.

## Obtendo Dados da API
O efeito `useEffect` é acionado ao carregar o componente ou quando o parâmetro `cripto` é alterado. Ele chama a função `getCoin` para buscar os dados da API.

### `getCoin()`
Esta função:
1. Faz uma requisição `fetch` para obter os dados da criptomoeda.
2. Se a resposta contiver um erro, redireciona para a página inicial.
3. Formata os valores da criptomoeda para uma melhor exibição.
4. Atualiza o estado `coin` com os dados formatados.
5. Define `loading` como `false` após o carregamento bem-sucedido.

```typescript
useEffect(() => {
    async function getCoin(){
        try{
            fetch(`https://api.coincap.io/v2/assets/${cripto}`)
            .then(response => response.json())
            .then((data: DataProps) => {
                if("error" in data){
                   navigate("/")
                   return;
                }

                const price = Intl.NumberFormat("en-US", {
                    style: "currency",
                    currency: "USD"
                })
    
                const priceCompact = Intl.NumberFormat("en-US", {
                    style: "currency",
                    currency: "USD",
                    notation: "compact"
                })

                const resultData = {
                    ...data.data,
                    formatedPrice: price.format(Number(data.data.priceUsd)),
                    formatedMarket: priceCompact.format(Number(data.data.marketCapUsd)),
                    formatedVolume: priceCompact.format(Number(data.data.volumeUsd24Hr)),
                }

                setCoin(resultData)
                setLoading(false)
            })
        } catch(err) {
            console.error(err);
            navigate("/")
        }
    }
    getCoin();
}, [cripto])
```

## Renderização do Componente
Caso os dados ainda estejam sendo carregados, o componente exibe uma mensagem de "Carregando...".
```tsx
if(loading  || !coin){
    return(
        <div className={styles.container}>
            <h2 className={styles.center}>Carregando...</h2>
        </div>
    )
}
```

Se os dados estiverem disponíveis, exibe os detalhes da criptomoeda:

### Estrutura Principal
```tsx
return(
    <div className={styles.container}>
        <h1 className={styles.center}>{coin?.name}</h1>
        <h1 className={styles.center}>{coin?.symbol}</h1>

        <section className={styles.content}>
            <img
                src={`https://assets.coincap.io/assets/icons/${coin?.symbol.toLowerCase()}@2x.png`}
                alt="Logo da moeda"
                className={styles.logo}
            />
            <h1>{coin?.name} | {coin?.symbol}</h1>

            <p><strong>Preço: {coin?.formatedPrice}</strong></p>
            
            <a>
                <strong>Mercado: {coin?.formatedMarket}</strong>
            </a>
            
            <a>
                <strong>Volume: {coin?.formatedVolume}</strong>
            </a>
            
            <a>
                <strong>Mudança 24H:</strong>
                <span className={Number(coin?.changePercent24Hr) > 0 ? styles.profit : styles.loss}>
                    {Number(coin?.changePercent24Hr).toFixed(2)}%
                </span>
            </a>
        </section>
    </div>
)
```

### Detalhes dos Elementos
1. **Nome e símbolo da criptomoeda**: Exibidos no topo da página.
2. **Imagem**: Obtida dinamicamente a partir do `symbol` da moeda.
3. **Preço**, **Mercado**, **Volume** e **Mudança em 24h**: Exibidos com formatação apropriada.
4. **Mudança de 24h**: A cor do texto varia dependendo do valor positivo ou negativo.

## Conclusão
O componente `Detail` obtém e exibe informações detalhadas de uma criptomoeda. Ele trata erros da API, formata os dados e apresenta uma interface amigável para o usuário. Caso haja algum erro na requisição, o usuário é redirecionado para a página inicial.

