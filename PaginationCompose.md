## Pagiação com PageLibrary

Adicione ao gradle

```sh
implementação "androidx.paging:paging-compose:1.0.0-alpha16"
```

```sh
RemoteMediator<Key : Any, Value : Any>
//É usado para carregar dados de forma incremental de uma fonte remota para uma fonte local.
```

```sh
PagingSource<Key: Any, Value: Any>
Define uma fonte de dados e como recuperar dados dessa fonte. Ele pode carregar dados de
qualquer fonte única, incluindo rede e bancos de dados locais.
```

```sh
class RecipesPagingSource(
    private val recipesRepository: RecipesRepository
) : PagingSource<Int, RecipeModel>() {

    override fun getRefreshKey(state: PagingState<Int, RecipeModel>): Int? {
        return state.anchorPosition
    }

    override suspend fun load(params: LoadParams<Int>): LoadResult<Int, RecipeModel> =
        try {
            val page = params.key ?: 0
            val size = params.loadSize
            val from = page * size
            val data = recipesRepository.getRecipes(from = from, size = size)
            if (params.placeholdersEnabled) {
                val itemsAfter = data.count - from + data.results.size
                LoadResult.Page(
                    data = data.results,
                    prevKey = if (page == 0) null else page - 1,
                    nextKey = if (data.results.isEmpty()) null else page + 1,
                    itemsAfter = if (itemsAfter > size) size else itemsAfter.toInt(),
                    itemsBefore = from
                )
            } else {
                LoadResult.Page(
                    data = data.results,
                    prevKey = if (page == 0) null else page - 1,
                    nextKey = if (data.results.isEmpty()) null else page + 1
                )
            }
        } catch (e: Exception) {
            LoadResult.Error(e)
        }
}
```


Nós estendemos PagingSourcepassando Intcomo um tipo de keye RecipeModelcomo um tipo de value.
RecipeModelé o que carregamos desta fonte.


> getRefreshKey— fornecer uma chave usada para o carregamento inicial 
> para o próximo PagingSourcedevido à invalidação desta PagingSource. 
> A chave é fornecida à carga via LoadParams.key. 
> O parâmetro da função, state: PagingState<Key, Value>,
> é o estado atual dos dados buscados, inclui páginas carregadas 
> de data( pages: List<Page<Key, Value>>), índice acessado mais recentemente
> na list( anchorPosition: Int?) e limites que foram dados ao inicializar o
> PagingDatastream( config: PagingConfig, falaremos mais sobre isso depois).


> load— acionar a carga assíncrona dos dados, do banco de dados ou da rede. 
> O parâmetro da função, params: LoadParams<Key>, são parâmetros para a solicitação
> de carregamento e contém o número solicitado de itens a carregar( loadSize: Int),
> se os placeholders estiverem habilitados( placeholdersEnabled: Boolean), e a chave 
> para a página a ser carregada( key: Int, explicada na getRefreshKeyfunção) . 
> O resultado dessa função é uma classe selada LoadResult<Key, Value>.

## LoadParamsé uma classe selada e tem três classes filhas:

- Refresh— que representa a solicitação de carregamento inicial
- Append— solicitação de carregamento que será anexada ao final da lista
- Prepend— solicitação de carregamento que será anexada ao início da lista

## LoadResulttambém tem três classes filhas. Aqui está o que eles são:

- Error— que representa o resultado do erro
- Invalid— que representa o resultado inválido
- Page— que representa o resultado bem-sucedido

## Na viewModel, estamos usando o Hilt para injeção de dependencia

```sh
@HiltViewModel
class MainViewModel @Inject constructor(
    private val recipesRepository: RecipesRepository
) : ViewModel() {

    val recipes: Flow<PagingData<RecipeModel>> = Pager(
        pagingSourceFactory = { RecipesPagingSource(recipesRepository) },
        config = PagingConfig(pageSize = 20)
    ).flow.cachedIn(viewModelScope)
}
``` 

> Em nosso ViewModel, criamos um fluxo de PagingData. 
> Pager é um construtor para um fluxo reativo de PagingData. 
> O construtor recebe três parâmetros:

 - config: PagingConfig— configuração do Paging. 
   Leva em um obrigatório e um par de parâmetros opcionais.
   Um parâmetro obrigatório é pageSize: Intqual é o número de itens a serem carregados
   de uma só vez de PagingSource. Alguns dos parâmetros opcionais que são interessantes 
   são enablePlaceholders: BooleanejumpThreshold: Int
 
- initialKey: Key— chave inicial doPagingSource

- pagingSourceFactory: () -> PagingSource<Key, Value>— fábrica lambda que 
  deve criar e retornar a PagingSourceinstância.

## MainScreen

```sh
@Composable
fun MainScreen(
    mainViewModel: MainViewModel = hiltViewModel()
) {
    val recipes = mainViewModel.recipes.collectAsLazyPagingItems()

    LazyColumn(
        modifier = Modifier
            .fillMaxSize()
            .padding(
                horizontal = 16.dp,
                vertical = 32.dp
            ),
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        item {
            Text(
                text = "Scroll for more recipes!",
                style = MaterialTheme.typography.h5
            )
            Spacer(modifier = Modifier.height(32.dp))
        }
        when (val state = recipes.loadState.prepend) {
            is LoadState.NotLoading -> Unit
            is LoadState.Loading -> {
                Loading()
            }
            is LoadState.Error -> {
                Error(message = state.error.message ?: "")
            }
        }
        when (val state = recipes.loadState.refresh) {
            is LoadState.NotLoading -> Unit
            is LoadState.Loading -> {
                Loading()
            }
            is LoadState.Error -> {
                Error(message = state.error.message ?: "")
            }
        }
        items(
            items = recipes,
            key = { it.id }
        ) {
            RecipeRow(recipeModel = it)
        }
        when (val state = recipes.loadState.append) {
            is LoadState.NotLoading -> Unit
            is LoadState.Loading -> {
                Loading()
            }
            is LoadState.Error -> {
                Error(message = state.error.message ?: "")
            }
        }
    }
}

@Composable
private fun RecipeRow(
    recipeModel: RecipeModel?
) {
    Spacer(modifier = Modifier.height(8.dp))
    Card(
        shape = RoundedCornerShape(16.dp),
        elevation = 8.dp
    ) {
        Row(
            modifier = Modifier
                .fillMaxWidth()
                .padding(16.dp),
            verticalAlignment = Alignment.CenterVertically
        ) {
            Image(
                painter = rememberAsyncImagePainter(recipeModel?.thumbnailUrl),
                contentDescription = recipeModel?.thumbnailAltText ?: "",
                modifier = Modifier
                    .placeholder(
                        visible = recipeModel == null,
                        highlight = PlaceholderHighlight.fade(),
                    )
                    .size(64.dp)
            )
            Spacer(modifier = Modifier.width(16.dp))
            Column(
                modifier = Modifier.weight(1f)
            ) {
                Text(
                    text = recipeModel?.name ?: "",
                    modifier = Modifier
                        .placeholder(
                            visible = recipeModel == null,
                            highlight = PlaceholderHighlight.fade(),
                        )
                        .fillMaxWidth()
                )
                Spacer(modifier = Modifier.height(8.dp))
                Text(
                    text = recipeModel?.getRating() ?: "",
                    style = MaterialTheme.typography.caption,
                    modifier = Modifier
                        .placeholder(
                            visible = recipeModel == null,
                            highlight = PlaceholderHighlight.fade(),
                        )
                        .fillMaxWidth()
                )
            }
        }
    }
    Spacer(modifier = Modifier.height(8.dp))
}

private fun LazyListScope.Loading() {
    item {
        CircularProgressIndicator(modifier = Modifier.padding(16.dp))
    }
}

private fun LazyListScope.Error(
    message: String
) {
    item {
        Text(
            text = message,
            style = MaterialTheme.typography.h6,
            color = MaterialTheme.colors.error
        )
    }
}
```





















