# Exerc-cio-Kidopi
# Descrição: O exercício consistirá na construção de um sistema (interface web) que possibilite ao usuário obter informações sobre os casos de mortes por Covid. Estes dados serão obtidos por meio da API-Covid-19 que está disponível no servidor da Kidopi. É possível obter dados do número de casos confirmados e mortes de vários países afetados pela COVID-19.

<!DOCTYPE html>   <!-- html5--> 
<html lang="en"> 
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dados de Covid</title> <!-- Titulo: Dados de Covid--> 
    <style> /* Estilos CSS */
        body { /* corpo */
            font-family: Verdana, sans-serif;
            font-size: 18px;
            margin: 0;
            padding: 0;
        }
        header, footer { /* cabeçalho e rodapé */
            background-color: #ffffff; /* #fffff = cor branca */
            font-size: 20px;
            padding: 10px;
            text-align: center;
        }
        main {
            padding: 20px;
        }
    </style>
</head>
<body> <!-- conteúdo que será vizualizado-->
    <header>
        <h1> Dados de Covid-19 </h1>
    </header>

    <main>
        <form method="GET" action="">
            <label for="país">Escolha um país:</label>
            <select name="país" id="país">
                <option value="Brazil">Brazil</option>
                <option value="Canada">Canada</option>
                <option value="Australia">Australia</option>
            </select>
            <button type="enviar">Consultar Dados</button>
        </form>

        <?php // Configuração do banco de dados usando php
        $servidor = "localhost";
        $nome_usuario = "root"; // nome de usuário do MySQL
        $senha = ""; // espaço para senha do MySQL, está vazia porque não tem senha 
        $nome_bd = "covid_data";

        // Conexão com o banco de dados
        $conexao = new mysqli($servidor, $nome_usuario, $senha, $nome_bd);

        // Verificação da conexão
        if ($conexao->connect_error) {
            die("A conexão falhou: " . $conexao->connect_error);
        }

        if (isset($_GET['país'])) {
            $pais = $_GET['país'];
            $url = "https://dev.kidopilabs.com.br/exercicio/covid.php?pais=" . $pais;

            // Comunicação com a API
            $resposta = file_get_contents($url); //resposta da API
            $dados = json_decode($resposta, true); //dados da resposta

            // Exibindo os dados retornados da API
            echo "<h2>Dados do (a) $pais</h2>"; //Título
            foreach ($dados as $estado) { // percorre os dados
                echo "<p>Estado: " . $estado['ProvinciaEstado'] . "</p>";
                echo "<p>Casos Confirmados: " . $estado['Confirmados'] . "</p>";
                echo "<p>Mortes: " . $estado['Mortos'] . "</p><hr>";
            }

            // Inserindo dados na tabela do MySQL
            $stmt = $conexao->prepare("INSERT INTO acesso_api (country, access_time) VALUES (?, NOW())");
            $stmt->bind_param("s", $pais);
            $stmt->execute();
            $stmt->close();
        }

        // Recuperar o último acesso
        $resultado = $conexao->query("SELECT country, access_time FROM acesso_api ORDER BY access_time DESC LIMIT 1"); 
        $ultimo_acesso = $resultado->fetch_assoc();
        $conexao->close();
        ?>
    </main>

    <footer> <!-- rodapé da página--> 
        <?php if ($ultimo_acesso): ?>
            <p> Ultimo acesso <?php echo $ultimo_acesso['country']; ?> on <?php echo $ultimo_acesso['access_time']; ?></p>
        <?php else: ?>
            <p>Nenhum acesso identificado.</p>
        <?php endif; ?>
    </footer>
</body>
</html>
