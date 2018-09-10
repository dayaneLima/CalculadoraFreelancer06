# CalculadoraFreelancer05

Se trata da continuação do app <a href="https://github.com/dayaneLima/CalculadoraFreelancer04">CalculadoraFreelancer04</a>

## Arrumando nosso repositório

Vamos agora arrumar o nosso repositório. Utilizaremos Injeção de dependência juntamente com o padrão Repository.

### Interface Repository

Dentro do projeto CalcFreelancer.Domain vamos criar uma pasta chamada Interfaces. Dentro desta pasta vamos criar uma interface chamada IRepository. 
Essa interface terá as assinaturas dos métodos básicos que é necessário para manipulação de dados. Primeiramente vamos criar a interface:

```c#
  public interface IRepository
  {
  }
````

Cada entidade terá seu repositório, então nossa interface poderá já definir que deverá ser de uma entidade específica, e utilizar dessa tipagem em seus métodos. Vamos então definir uma tipagem genérica que depois em nossos repositórios específicos irá definir a tipagem concreta.
Então mude o nome da interface dessa forma:

```c#
  public interface IRepository<TEntity>
  {
  }
````

Agora vamos criar as funções básicas:

```c#
    public interface IRepository<TEntity>
    {
        Task<TEntity> Find(string id);
        Task<TEntity> GetFirst();
        Task<IEnumerable<TEntity>> GetAll();
        Task Insert(TEntity tEntity);
        Task Update(TEntity tEntity);
        Task Delete(TEntity tEntity);
    }
  ````
  
  ### Interface IProfissionalRepository
  
Agora que temos nossa interface genérica, vamos criar uma interface para cada domínio que temos. Ainda dentro do projeto CalcFreelancer.Domain, na pasta Profissionais crie uma pasta chamada Repository, e dentro dela crie uma interface chamada IProfissionalRepository, dessa forma:
  
```c#
 public interface IProfissionalRepository
  {
  }
````
  
Como temos uma interface base, chamada IRepository, isso significa que todas as outras interfaces de repositório devem implementar dela, pois ela possui as assinaturas das operações base. Então vamos implementar de IRepository.
    
```c#
 public interface IProfissionalRepository : IRepository
  {
  }
````

Lembra que nossa interface IRepository tem uma tipagem (IRepository<TEntity>) ? É agora que vamos utilizar isso e ficará mais fácil de entender.
	
Como estamos criando agora o repositório do Profissional, essa tipagem do repositório será do Profissional, ou seja, agora todas aquelas assinaturas de funções retornarão e receberão objetos do tipo Profissional.

A interface então ficará dessa forma:

```c#
    public interface IProfissionalRepository : IRepository<Profissional>
    {
    }
````
 
 Como ainda não temos nenhuma operação específica para o Profissional, nosso repositório ficará como mostrado acima.
 
   ### Interface IProjetoRepository
 
Agora vamos fazer o mesmo para o Projeto. Ainda no projeto CalcFreelancer.Domain, dentro da pasta Projetos crie uma pasta chamada Repository. Dentro dela crie uma interface chamada IProjetoRepository, implemente de IRepository definindo a tipagem para Projeto. 

Ficará assim:
 
 ```c#
public interface IProjetoRepository : IRepository<Projeto>
{
}
````

Perfeito, agora temos em nosso domínio todas as assinaturas base para manipulação de dados. O reuso e o ganho com isso é enorme, pois agora como há as operações base, quem desejar implementar o acesso a dados para o nosso domínio basta implementar dessas interfaces. 
Nossa camada de domínio não precisa saber como o acesso a dados é feito, se é MySql, Azure, SQLite, arquivo de texto ou qualquer outra tecnologia, ele só se importa em chamar os métodos definidos na interface e receber o resultado especificado nela.

## Camada CalcFreelancer.Infra.Data

A implementação do acesso a dados é feito na camada CalcFreelancer.Infra.Data, vamos corrigí-la.

### Repositório Base

A AzureRepository será a nossa classe base e genérica. Então primeiramente ela terá uma tipagem genérica relacionada a ela, como fizemos no IRepository. Ficará assim:

```c#
  public class AzureRepository<TEntity>
  {
      ...
  }
````

Como  ela será a nossa genérica, ela poderá implementar de IRepository, dessa forma:

```c#
 public class AzureRepository<TEntity> : IRepository<TEntity>
 {
    ...
 }
````

Todas as nossas Entidades herdam de Entity, podemos então definir em nossa classe que AzureRepository que o TEntity será do tipo Entity,  dessa forma:

```c#
    public class AzureRepository<TEntity> : IRepository<TEntity> where TEntity : Entity
    {
        ...
    }
````

Agora que corrigimos a assinatura da classe, vamos corrir suas propriedades e métodos.
Primeiramente a nossa propriedade Table não será mais do tipo Profissional e sim do nosso tipo genérico TEntity, ficando dessa forma:


```c#
     private IMobileServiceTable<TEntity> Table;
````
 
 No construtor da classe AzureRepository onde instanciamos essa propriedade, devemos corrigir também a tipagem. No lugar de Profissional vamos trocar por TEntity:
 
 ```c#
   public AzureRepository()
  {
      string MyAppServiceURL = "sua url aqui";
      Client = new MobileServiceClient(MyAppServiceURL);
      Table = Client.GetTable<TEntity>();
  }
````
 
 Vamos as correções dos métodos agora. Primeiramente altere os métodos Insert e Delete, o tipo de retorno deles é void, mude para Task, pois será operações assíncronas.
 
Agora tudo que é do tipo Profissional deverá ser alterado para ser do tipo TEntity, vamos fazer manualmente para entendermos melhor, alterando método a método. A classe ficou assim:

```c#
    public class AzureRepository<TEntity> : IRepository<TEntity> where TEntity : Entity
    {
        private IMobileServiceClient Client;
        private IMobileServiceTable<TEntity> Table;

        public AzureRepository()
        {
            string MyAppServiceURL = "rua url aqui";
            Client = new MobileServiceClient(MyAppServiceURL);
            Table = Client.GetTable<TEntity>();
        }

        public async Task<IEnumerable<TEntity>> GetAll()
        {
            var empty = new TEntity[0];
            try
            {
                return await Table.ToEnumerableAsync();
            }
            catch (Exception)
            {
                return empty;
            }
        }

        public async Task Insert(TEntity tEntity)
        {
            await Table.InsertAsync(tEntity);
        }

        public async Task Update(TEntity tEntity)
        {
            await Table.UpdateAsync(tEntity);
        }

        public async Task Delete(TEntity tEntity)
        {
            await Table.DeleteAsync(tEntity);
        }

        public async Task<TEntity> Find(string id)
        {
            var itens = await Table.Where(i => i.Id == id).ToListAsync();
            return (itens.Count > 0) ? itens[0] : null;
        }

        public async Task<TEntity> GetFirst()
        {
            var itens = await Table.ToListAsync();
            return (itens.Count > 0) ? itens[0] : null;
        }
    }

````

### ProfissionalRepository

Vamos agora criar uma classe para implementar de nossa interface IProfissionalRepository. Ainda dentro da camada CalcFreelancer.Infra.Data, dentro da pasta Repository crie uma classe chamada ProfissionalRepository.

```c#
 public class ProfissionalRepository
 {
 }
````

Agora ela deverá herdar da nossa classe de repositório base, a AzureRepository, que contém todas as operações base já implementadas. 

```c#
 public class ProfissionalRepository : AzureRepository
 {
 }
````

A AzureRepository também espera uma tipagem, pois a mesma é genérica, como estamos no repositório do Profissional, informaremos a classe que será do tipo Profissional.

```c#
 public class ProfissionalRepository : AzureRepository<Profissional>
 {
 }
````

Por fim temos que implementar da interface do repositório do profissional, a IProfissionalRepository, dessa forma:

```c#
 public class ProfissionalRepository : AzureRepository<Profissional>, IProfissionalRepository
  {
  }
````

Prontinho, nosso repositório do profissional está criado.

### ProjetoRepository

Vamos fazer na ProjetoRepository o mesmo que foi feito na ProfissionalRepository. Ainda dentro da camada CalcFreelancer.Infra.Data, 
dentro da pasta Repository crie uma classe chamada ProjetoRepository.

A classe deverá herdar do AzureRepository, informando a tipagem Projeto e deverá implementar da interface IProjetoRepository. Ficará dessa forma:

```c#
  public class ProjetoRepository : AzureRepository<Projeto>, IProjetoRepository
  {
  }
````

Pronto, nossa camada de Infra está corrigida.

## Camada Principal - CalcFreelancer

Falta agora alterar a nossa camada principal. 

### Criando contrato para os serviços

Dentro da pasta Services crie uma pasta chamada Interfaces. Vamos agora criar uma interface para o nosso ProfissionalService chamada IProfissionalService.

```c#
public interface IProfissionalService
{
}
````

Vamos criar agora as assinaturas dos métodos. Inicialmente como temos somente a operaçáo de inserir, vamos criar a assinatura deste método.

```c#
public interface IProfissionalService
{
	void Inserir(Profissional profissional);
}
````

Agora a a nossa classe ProfissionalService deverá herdar de IProfissionalService, ficando desta forma:

```c#
public class ProfissionalService : IProfissionalService
{
        private readonly AzureRepository ProfissionalRepository;

        public ProfissionalService()
        {
            ProfissionalRepository  = new AzureRepository();
        }

        public void Inserir(Profissional profissional)
        {
            ProfissionalRepository.Insert(profissional);
        }
}
````

Vamos fazer a mesma coisa para o ProjetoService. Vamos crair dentro da pasta Services/Interfaces, uma interface chamada IProjetoService que terá também somente a assinatura do método de inserir. Ficará assim:

```c#
public interface IProjetoService
{
	void Inserir(Projeto projeto);
}
````

Agora a a nossa classe ProjetoService deverá herdar de IProjetoService, ficando desta forma:

```c#
public class ProjetoService: IProjetoService
{
        private readonly AzureProjetoRepository ProjetoRepository;

        public ProjetoService()
        {
            ProjetoRepository = new AzureProjetoRepository();
        }

        public void Inserir(Projeto projeto)
        {
            ProjetoRepository.Insert(projeto);
        }
}
````

### Injeção de dependência - Instalação e configuração do Unity

Primeiramente não vamos instanciar objetos do tipo dos nossos repositórios nem dos nossos serviços, vamos fazer da forma correta, para reduzir nosso acoplamento, vamos utilizar a injeção de dependência.

Vamos instalar uma biblioteca para cuidar de resolver as dependências, ela será nosso Builder e tratará nosso container, conceitos que vimos na aula. 

Uma das bibliotecas bastante utilizada é a Unity. Vamos então na nossa Solution, clicar com o botão direito e ir em Manage NuGet Packages for solution, em Browse pesquise por Unity. Após encontrá-la escolha para instalar no projeto principal, no do Android e IOS, e então clique em install.

 <img src="https://github.com/dayaneLima/CalculadoraFreelancer05/blob/master/Docs/Imgs/aula_05_install_unity_01.png" alt="Instalação Unity" width="100%">

Após a instalação vamos registrar nossos containers, ou seja, informar se for a interface IProjetoRepository, deverá entregar uma instância do tipo ProjetoRepository. Edite o arquivo chamado App.xaml.cs que se encontra em nosso projeto principal.

Em seu construtor, após a funcão InitializeComponent(), instâncie um objeto do tipo UnityContainer, dessa forma:

```c#
public partial class App : Application
{
	public App ()
	{
	      	InitializeComponent();

            	var unityContainer = new UnityContainer();

            	MainPage = new NavigationPage(new ProjetoPage());
	}
    
    ...
    
}
````

Agora vamos registrar as nossas interfaces informando qual classe deverá ser instânciada ao se esperar um objeto da tipagem da interface.

```c#
public partial class App : Application
{
	public App ()
	{
		InitializeComponent();

		var unityContainer = new UnityContainer();

		unityContainer.RegisterType<IProjetoRepository, ProjetoRepository>();
		unityContainer.RegisterType<IProfissionalRepository, ProfissionalRepository>();

		unityContainer.RegisterType<IProjetoService, ProjetoService>();
		unityContainer.RegisterType<IProfissionalService, ProfissionalService>();

		MainPage = new NavigationPage(new HomePage());
	}
}
````

Vamos agora informar ao Builder que deverá utilizar esse container que acabamos de definir, dessa forma:

```c#
public partial class App : Application
{
	public App ()
	{
		InitializeComponent();

		var unityContainer = new UnityContainer();

		unityContainer.RegisterType<IProjetoRepository, ProjetoRepository>();
		unityContainer.RegisterType<IProfissionalRepository, ProfissionalRepository>();

		unityContainer.RegisterType<IProjetoService, ProjetoService>();
		unityContainer.RegisterType<IProfissionalService, ProfissionalService>();

		ServiceLocator.SetLocatorProvider(() => new UnityServiceLocator(unityContainer));

		MainPage = new NavigationPage(new HomePage());
	}

}
````

### Injeção de dependência - ViewModels

Vamos corrigir as nossas ViewModels para utilização de interfaces ao invés de classes concretas. Vamos utilizar injeção de dependência via construtor.

Edite a CalculoValorHoraPageViewModel.cs. A nossa propriedade do tipo ProfissionalService será alterada para o tipo IProfissionalService.

```c#
private readonly IProfissionalService ProfissionalService;
````

Vá no construtor dessa classe e coloque nos parâmetros recebidos um objeto do tipo IProfissionalService. Ao invés de instanciar a nossa propriedade ProfissionalService vamos atribuir a ela o valor recebido por parâmetro.

```c#
	...

	private readonly IProfissionalService ProfissionalService;

	public CalculoValorHoraPageViewModel(IProfissionalService profissionalService)
	{
		GravarCommand = new Command(ExecuteGravarCommand);
		Profissional = new Profissional();
		ProfissionalService = profissionalService;
	}
	
	...
	
````

Agora vamos fazer o mesmo para a ViewModel ProjetoPageViewModel. Vamos alterar a tipagem da propriedade ProjetoService para IProjetoService. Vamos receber no construtor da classe ProjetoPageViewModel um objeto do tipo IProjetoService e vamos atribuí-lo a nossa propriedade ProjetoService:

```c#
	...
	
	private readonly IProjetoService ProjetoService;

	public ProjetoPageViewModel(IProjetoService projetoService)
	{
		GravarCommand = new Command(ExecuteGravarCommand);
		LimparCommand = new Command(ExecuteLimparCommand);
		ProjetoService = projetoService;
	}
	
	...
````

### Injeção de dependência - Services

Agora edite o arquivo chamado ProfissionalService. Nossa propriedade que era do tipo AzureRepository será agora do tipo IProfissionalRepository. Então como fizemos nas ViewModels, vamos receber via construtor um objeto do tipo IProfissionalRepository e atribuir a nossa propriedade ProfissionalRepository.

```c#
	...

	private readonly IProfissionalRepository ProfissionalRepository;

	public ProfissionalService(IProfissionalRepository profissionalRepository)
	{
		ProfissionalRepository  = profissionalRepository;
	}

	...

````

Agora faça o  mesmo com o ProjetoService. Vamos alterar a tipagem do ProjetoRepository para IProjetoRepository, receber um objeto dessa tipagem no construtor e atribuir a nossa propriedade.

```c#

	...

	private readonly IProjetoRepository ProjetoRepository;

	public ProjetoService(IProjetoRepository projetoRepository)
	{
		ProjetoRepository = projetoRepository;
	}

	...

````

Como não usaremos mais a classe AzureProjetoRepository  que está dentro do projeto CalcCalcFreelancer.Infra.Data, vamos excluí-lo.

### Injeção de dependência - Views

Agora se abrirmos o arquivo CalculoValorHoraPage.xaml.cs, no construtor onde instanciavamos um objeto do tipo CalculoValorHoraPageViewModel estará marcado com erro, pois a nossa ViewModel espera receber um objeto do tipo IProfissionalService.
Mas não podemos criar uma instância concreta de ProfissionalService e passar por parâmetro, pois estariamos acabando com a vantagem da injeção de dependência e do reuso. Temos que chamar o nosso Builder, informando para ele criar uma instância do tipo CalculoValorHoraPageViewModel, e ele será responsável por resolver as dependências. Ficará assim:

```c#
... 

public CalculoValorHoraPage ()
{
	InitializeComponent ();
	var viewModel = ServiceLocator.Current.GetInstance<CalculoValorHoraPageViewModel>();
	BindingContext = viewModel;
}   

...

````

Vamos fazer o mesmo com o arquivo ProjetoPage.xaml.cs. Ao invés de instanciar um objeto do tipo ProjetoPageViewModel, vamos solicitar ao nosso Builder que nos retorne uma instância desse tipo.

```c#

...

public ProjetoPage ()
{
	InitializeComponent ();
	var viewModel = ServiceLocator.Current.GetInstance<ProjetoPageViewModel>();
	BindingContext = viewModel;
}
	
...

````

## Resultado

Agora temos um código mais organizado, limpo, com um reuso bem maior do que anteriormente. A aparência do app não mudou em nada, mas internamente obtivemos um ganho enorme, facilitando a nossa vida de desenvolvedor, no momento de dar manutenções e reutilização de código.

## Trocando nossa camada de CalcFreelancer.Infra.Data

Pra mostrar a grande vantagem do uso da arquitetura em camadas juntamente com a injeção de dependência vamos alterar um pouco nosso armazenamento de dados. Vamos supor o seguinte cenário: O cliente não quer mais utilizar o Azure, pois está gastando muito e analisou que basta as informações do aplicativo ficarem salvas no próprio dispositivo que já será o suficiente. Para resolver essa nova solicitação do cliente, vamos utilizar ao invés do Azure o SqLite.


### Instalação do SqLite

Clique com o botão diteiro sobre a Soluction, vá em Manage Nuget Packages for Solution... e clique em na guia Browse. Pesquise pela biblioteca chamada sqlite-net-pcl. Ao encontrá-la instale em todos os projetos.


 <img src="https://github.com/dayaneLima/CalculadoraFreelancer05/blob/master/Docs/Imgs/aula_05_install_sqlite.png" alt="Instalação SqLite" width="100%">
 
 ### Caminho do Banco de Dados
 
 O Android e o IOS tem sistemas de armazenamento diferentes, o caminho do banco de dados em cada um também será. Então pela primeira vez precisaremos criar uma interface com o método que retorna o caminho do banco de dados no nosso projeto compartilhado, e criar a implementação dessa interface em cada plataforma.
 
 #### Interface IDatabaseFile
 
 No projeto CalcFreelancer.Domain, na pasta Interfaces, crie uma chamada IDatabaseFile. Essa interface terá uma função que receberá o nome do banco de dados e retonará o caminho completo para sua criação e acesso. A interface ficou assim:
 
 ```c#
public interface IDatabaseFile
{
	string GetFilePath(string file);
}
 ````
 
 #### Implementando IDatabaseFile no projeto Android
 
 No projeto CalcFreelancer.Android crie uma pasta chamada Database. Dentro desta pasta crie uma classe chamada DatabaseFile.
 
 ```c#
    public class DatabaseFile
    {
       
    }
 ````

Agora vamos implementar da interface IDatabaseFile:

```c#
public class DatabaseFile : IDatabaseFile
{

}
````

A interface IDatabaseFile está no projeto CalcFreelancer.Domain, então temos que adicionar a referência desse projeto no nosso CalcFreelancer.Android. No projeto CalcFreelancer.Android clique com o botão direito em References, vá em Add Reference, Clique sobre Projects e além do CalcFreelancer que já está marcado, marque também o CalcFreelancer.Domain.


 <img src="https://github.com/dayaneLima/CalculadoraFreelancer05/blob/master/Docs/Imgs/aula_05_add_reference_domain_android.png" alt="Adicionar Referência no projeto Android" width="100%">

Agora ao clicar em ctrl + . sobre o IDatabaseFile que está marcado com erro o Visual Studio nos dá a opção de dar o using no projeto CalcFreelancer.Domain.Interfaces.

Agora vamos implementar a função GetFilePath. Vamos salvar no caminho das pastas de sistema utilizando o SpecialFolder, escolhendo o Personal, que é um repositório para documentos.

```c#
public class DatabaseFile : IDatabaseFile
{
	public string GetFilePath(string filename)
	{
	    string path = System.Environment.GetFolderPath(System.Environment.SpecialFolder.Personal);	    
	}
}
````

Agora vamos utilizar o Path.Combine para concatenar nosso nome do banco de dados que passamos por parâmetro com o caminho da pasta que desejamos salvar.

```c#
public class DatabaseFile : IDatabaseFile
{
	public string GetFilePath(string filename)
	{
	    string path = System.Environment.GetFolderPath(System.Environment.SpecialFolder.Personal);
	    return Path.Combine(path, filename);
	}
}
````

Agora temos que informar que a nossa classe DatabaseFile irá substituir a interface IDatabaseFile do nosso projeto principal utilizando o serviço de dependência do Xamarin Forms. Para isso, em cima do namespace vamos adicionar o seguinte trecho de código:

```c#
[assembly: Dependency(typeof(DatabaseFile))]
````

Ao clicar em ctro + . dê o using no Xamarin.Forms, o Visual Studio te dará outras opções.

Ficou assim nossa classe no Android:


```c#
[assembly: Dependency(typeof(DatabaseFile))]
namespace CalcFreelancer.Droid.Database
{
    public class DatabaseFile : IDatabaseFile
    {
        public string GetFilePath(string filename)
        {
            string path = System.Environment.GetFolderPath(System.Environment.SpecialFolder.Personal);
            return Path.Combine(path, filename);
        }
    }
}
````


#### Implementando IDatabaseFile no projeto IOS

Vamos fazer similar ao que fizemos no projeto Android. Vá no projeto CalcFreelancer.IOS, crie uma pasta chamada Database, dentro dela crie uma classe chamada DatabaseFile que implemente de IDatabaseFile.

```c#
public class DatabaseFile: IDatabaseFile
{
}
````

Adicione a referência ao projeto CalcFreelancer.Domain como fizemos no Android (clique com o botão direito em References, vá em Add Reference, Clique sobre Projects e além do CalcFreelancer que já está marcado, marque também o CalcFreelancer.Domain).

Agora ao clicar em ctrl + . sobre o IDatabaseFile que está marcado com erro o Visual Studio nos dá a opção de dar o using no projeto CalcFreelancer.Domain.Interfaces.

Vamos implementar a função GetFilePath. Ela é bem similar ao que fizemos no projeto Android, mas temos que armazenar na pasta Library para não ocorrer erro no IOS:

```c#
public class DatabaseFile: IDatabaseFile
{
	public string GetFilePath(string file)
	{
	    string docFolder = Environment.GetFolderPath(Environment.SpecialFolder.Personal);
	    string libFolder = System.IO.Path.Combine(docFolder, "..", "Library");
	    return Path.Combine(libFolder, file);
	}
}
````

Como fizemos no projeto do Android, temos que informar que a nossa classe DatabaseFile irá substituir a interface IDatabaseFile do nosso projeto principal utilizando o serviço de dependência do Xamarin Forms. Para isso, em cima do namespace vamos adicionar o seguinte trecho de código:

```c#
[assembly: Dependency(typeof(DatabaseFile))]
````

Ao clicar em ctro + . dê o using no Xamarin.Forms, o Visual Studio te dará outras opções.

Ficou assim nossa classe no IOS:

```c#
[assembly: Dependency(typeof(DatabaseFile))]
namespace CalcFreelancer.iOS.Database
{
    public class DatabaseFile: IDatabaseFile
    {
        public string GetFilePath(string file)
        {
            string docFolder = Environment.GetFolderPath(Environment.SpecialFolder.Personal);
            string libFolder = System.IO.Path.Combine(docFolder, "..", "Library");
            return Path.Combine(libFolder, file);
        }
    }
}
````

#### Implementado a classe DatabaseContext - projeto CalcFreelancer.Infra.Data

Vamos agora no projeto CalcFreelancer.Infra.Data e implementar o acesso a dados com o SqLite. 

Primeiramente dentro da pasta Repository crie uma classe chamada DatabaseContext que será responsável por criar nossas tabelas no banco de dados.

```c#
public class DatabaseContext
{
}
````

Essa classe terá duas propriedades, uma para representar a nossa própria classe DatabaseContext, pois utilizaremos o padrão Singleton, e outra para representar nossa conexão com o banco de dados:

```c#
public class DatabaseContext
{
	static DatabaseContext DatabaseSingleton;
	static SQLiteAsyncConnection Conn;
}
````

Vamos agora criar uma função chamada Database que retornará a conexão com o SqLite, e nela verificamos se nossa propriedade DatabaseSingleton é nula, caso for, a intânciamos, garantindo que vamos instanciá-la somente uma vez:

```c#
    ...

        public static SQLiteAsyncConnection Database
        {
            get
            {
                if (DatabaseSingleton == null)
                    DatabaseSingleton = new DatabaseContext();

                return Conn;
            }
        }

    ...
````

Agora temos que instalar o Xamarin.Forms também no nosso projeto CalcFreelancer.Infra.Data. Como o Xamarin Forms costuma ter atualizações, temos que instalar com a mesma versão que estamos utilizando nos outros projetos.

Clique com o botão direito na Soluction e vá em Manage Nuget Packages For Solution, em Installed procure Xamarin.Forms. Marque o projeto CalcFreelancer.Infra.Data e em Version escolha a mesma que já está instalada nos outros projetos (basta olhar a versão que está escrita no campo Installed. Após escolher a versão correta clique em install.

 <img src="https://github.com/dayaneLima/CalculadoraFreelancer05/blob/master/Docs/Imgs/aula_05_instalacao_xamarin_forms_infra.png" alt="Instalar o Xamarin Forms no projeto Infra.Data" width="100%">

Vamos criar um construtor para a nossa classe DatabaseContext:

```c#
public DatabaseContext()
{

}
````

No construtor devemos obter o caminho do banco de dados que especificamos dentro de cada plataforma. Vamos utilizar DependencyService do Xamarin forms para nos dar uma instância do tipo IDatabaseFile que implementamos tanto no projeto IOS quando no Android. E após obter essa instância podemos chamar a função GetFilePath, passando para ela o nome do banco de dados que desejamos. Vamos chamar o banco de dados de calcfreelancer.db3. Ficará assim:

```c#
public DatabaseContext()
{
    var caminhoBancoDeDados = DependencyService.Get<IDatabaseFile>().GetFilePath("calcfreelancer.db3");
}
````

Agora vamos criar a conexão com o SqLite passando para ela o caminho do banco de dados:

```c#
 public DatabaseContext()
{
    var caminhoBancoDeDados = DependencyService.Get<IDatabaseFile>().GetFilePath("calcfreelancer.db3");
    Conn = new SQLiteAsyncConnection(caminhoBancoDeDados);
}
````

Por último vamos pedir ao SqLite que crie nossas tabelas de Projeto e Profissional:

```c#
 public DatabaseContext()
{
    var caminhoBancoDeDados = DependencyService.Get<IDatabaseFile>().GetFilePath("calcfreelancer.db3");
    Conn = new SQLiteAsyncConnection(caminhoBancoDeDados);

    Conn.CreateTableAsync<Projeto>().Wait();
    Conn.CreateTableAsync<Profissional>().Wait();
}
````

Nossa classe final ficou assim:

```c#
public class DatabaseContext
{
	static DatabaseContext DatabaseSingleton;
	static SQLiteAsyncConnection Conn;

	public static SQLiteAsyncConnection Database
	{
	    get
	    {
		if (DatabaseSingleton == null)
		    DatabaseSingleton = new DatabaseContext();

		return Conn;
	    }
	}

	public DatabaseContext()
	{
	    var caminhoBancoDeDados = DependencyService.Get<IDatabaseFile>().GetFilePath("calcfreelancer.db3");
	    Conn = new SQLiteAsyncConnection(caminhoBancoDeDados);

	    Conn.CreateTableAsync<Projeto>().Wait();
	    Conn.CreateTableAsync<Profissional>().Wait();
	}
}
````

#### Implementado a classe SqLiteRepository - projeto CalcFreelancer.Infra.Data

Ao invés de alterarmos a classe AzureRepository, vamos criar outra, chamada SqLiteRepository, dentro da pasta Repository. Ela terá a mesma lógica da AzureRepository, utilizando a tipagem genérica e implementando de IRepository:

```c#
public class SqLiteRepository<TEntity> : IRepository<TEntity> where TEntity : Entity
{
}
````

Vamos adicionar após o Entity o new(), pois o SqLite exige que o Entity seja uma classe com construtor instânciável, ficando assim:

```c#
public class SqLiteRepository<TEntity> : IRepository<TEntity> where TEntity : Entity, new()
{
}
````

Agora vamos implementar todas as funções do IRepository, nossa classe então ficou assim:

```c#
public class SqLiteRepository<TEntity> : IRepository<TEntity> where TEntity : Entity, new()
{
	public async Task Delete(TEntity tEntity)
	{
	    await DatabaseContext.Database.DeleteAsync(tEntity);
	}

	public async Task<TEntity> Find(string id)
	{
	    return await DatabaseContext.Database.Table<TEntity>().Where(x => x.Id == id).FirstAsync();
	}

	public async Task<IEnumerable<TEntity>> GetAll()
	{
	    return await DatabaseContext.Database.Table<TEntity>().ToListAsync();
	}

	public async Task<TEntity> GetFirst()
	{
	    return await DatabaseContext.Database.Table<TEntity>().FirstOrDefaultAsync();
	}

	public async Task Insert(TEntity tEntity)
	{
	    await DatabaseContext.Database.InsertAsync(tEntity);
	}

	public async Task Update(TEntity tEntity)
	{
	    await DatabaseContext.Database.UpdateAsync(tEntity);
	}
}
````


#### Alterando os repositórios ProfissionalRepository e ProjetoRepository - projeto CalcFreelancer.Infra.Data

Agora nas classes ProfissionalRepository.cs e ProjetoRepository.cs ao invés de herdarmos de AzureRepository,  vamos trocar por SqLiteRepository:

```c#
public class ProjetoRepository : SqLiteRepository<Projeto>, IProjetoRepository
{
}
````

```c#
public class ProfissionalRepository : SqLiteRepository<Profissional>, IProfissionalRepository
{
}
````

#### Alterando nosso domínio - projeto CalcFreelancer.Domain

Vamos agora fazer umas pequenas alterações em nosso domínio. No projeto CalcFreelancer.Domain, nas classes Profissional e Projeto, utilizavamos o DataAnotation da biblioteca Microsoft.WindowsAzure.MobileServices para falar qual o nome da nossa tabela do banco de dados, agora temos que utilizar os DataAnotation do SqLite. 

Edite o arquivo Profissional.cs dentro do projeto CalcFreelancer.Domain, dentro da pasta Profissionais. Onde estavamos utilizando o DataTable, troque para Table e dê o using no SQLite:

```c#
[Table("Profissional")]
public class Profissional : Entity
{
	public double ValorGanhoMes { get; set; }
	public int HorasTrabalhadasPorDia { get; set; }
	public int DiasTrabalhadosPorMes { get; set; }
	public int DiasFeriasPorAno { get; set; }
	public double ValorPorHora { get; set; }
}
````

Faça o mesmo para o Projeto.cs:

```c#
[Table("Projeto")]
public class Projeto : Entity
{
	public string Nome { get; set; }
	public double ValorPorHora { get; set; }
	public int HorasPorDia { get; set; }
	public int DiasDuracaoProjeto { get; set; }
	public double ValorTotal { get; set; }
}
````

#### Alterando nosso domínio base - projeto CalcFreelancer.Domain.Core

Vamos alterar a nossa classe base Entity, pois a mesma utilizava também da biblioteca Microsoft.WindowsAzure.MobileServices e agora temos que utilizar do SQLite. Como não vamos utilizar mais o Version do Azure, podemos retirá-lo:

```c#
public class Entity
{
	public string Id { get; set; }
	public DateTime CreatedAt { get; set; }
	public DateTime UpdatedAt { get; set; }
}
````

No SQLite, temos que informar quem serão as chaves primárias de nossas tabelas, no caso será nossa propriedade Id. Vamos utilizar o DataAnotation para informar que nossa propriedade Id será Primary Key. O Azure criava o Id para nós automaticamente, como não estamos utilizando mais o Azure, vamos adicionar um construtor para nossa classe Entity e atribuir um valor para a propriedade Id. Para isso vamos utilizar o Guid do c#, que gera um hash único. Ficou assim:

```c#
public class Entity
{
	[PrimaryKey]
	public string Id { get; set; }
	public DateTime CreatedAt { get; set; }
	public DateTime UpdatedAt { get; set; }

	public Entity()
	{
	    Id = Guid.NewGuid().ToString();
	}
}
````

## Resultado

Prontinho, agora alteramos o nosso projeto para usar o SQLite. Deu um pouquinho de trabalho, mas tenha certeza de que sem a injeção de dependência e utilização de uma boa arquitetura, teria nos dado muito mais trabalho. Alteramos apenas nossas camadas de Domain e Infra, além do Android e IOS para acesso ao caminho do bando de dados.
