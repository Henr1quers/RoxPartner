<h1 align="center"> RoxPartner Teste </h1>

<p>Esse é um teste com o objetivo de conhecer um pouco mais sobre a forma de trabalhar com dados e a resolução de problemas que envolvem engenharia de dados. Analisando os seguintes aspectos: </p>
<ul>
  <li>Fazer a modelagem conceitual dos dados;</li>
  <li>Criação da infraestrutura necessária</li>
  <li>Criação de todos os artefatos necessários para carregar os arquivos para o banco criado;</li>
  <li>Desenvolvimento de SCRIPT para análise de dados;</li>
   <li>Criar um relatório em qualquer ferramenta de visualização de dados. (OPCIONAL) </li>
</ul>

<h2> Modelo Fisio </h2>

![image](https://user-images.githubusercontent.com/92615548/183257472-2a9fff50-8c15-4051-a0ea-c2d7d2acab79.png)

<p>Para e resolução desse projeto foi escolhido a plataforma Microsoft Azure para criar a infraestrutura necessária pelo custo-benefício e por oferecem no primeiro cadastro realizado uma quantia para utilizar dentro da plataforma. Foi utilizado no projeto o Banco de dados SQL utilizando a camada de preço (Uso Geral - Sem servidor: Gen5, 1 vCore) , que atende perfeitamente a necessidade para o projeto. </p>

![image](https://user-images.githubusercontent.com/92615548/183257381-1b5cc99c-b3f6-4f2b-8d5d-81fb2625d0ae.png)

<p>Depois de analisar todos os arquivos CSV para serem importado para o banco de dados, ficou decidido que seria criado um banco de dados com o nome RoxPartner e dentro do mesmo seria criado Schemas e sua tabelas relacionadas, como mostrado a baixo:</p>

<ul>
  <li>
    Person (Schema)
    <ul>
      <li>Person (Table)</li>
    </ul>
  </li>
  <li>
    Production (Schema)
    <ul>
      <li>Product (Table)</li>
    </ul>
  </li>
  <li>
    Production (Schema)
    <ul>
      <li>Customer (Table)</li>
      <li>SalesOrderDetail (Table)</li>
      <li> SalesOrderHeader (Table)</li>
      <li>SpecialOfferProduct (Table)</li>
    </ul>
  </li>
</ul>

