redesocial
=========
<?php 
require_once('startsession.php');
require_once('conecta.php');
require_once('image_up.php');
require_once('header.php');
require_once('menunav.php');
if(!isset($_SESSION['id'])){
	echo '<div id="site_content">';	
	echo '<div id="sidebar_container">';
	echo '<div class="sidebar">';
	echo '<h3>Página Principal</h3>';
	echo '<p class="sucess">Entre com Login e Senha</p>';
	echo '<p class="sucess">Para acessar o seu perfil logo acima</p>';
	echo '<p class="sucess">Ou clique em cadastrar para criar o seu perfil</p>';
	echo '</div>';//fim sidebar_container
	echo '</div>';//fim sidebar
}
else{
echo '<div id="site_content">';	
	echo '<div id="sidebar_container">';
	echo '<div class="sidebar">';
	echo '<h3>Altere o seu Perfil</h3>';
	echo '<p class="sucess">Coloque seus dados pessoais</p>';
	echo '<p class="sucess">e inclua ou altere sua foto</p>';
	echo '</div>';//fim sidebar_container
	echo '</div>';//fim sidebar
}
require_once('conteudo.php');

// Conexão com o banco de dados
  $dbc = mysqli_connect(DB_HOST, DB_USER, DB_PASSWORD, DB_NAME) or die(mysqli_error());

  if (isset($_POST['submit'])) {
    // pega os dados em POST
    $nome = mysqli_real_escape_string($dbc, trim($_POST['nome']));
    $sobrenome = mysqli_real_escape_string($dbc, trim($_POST['sobrenome']));
    $sexo = mysqli_real_escape_string($dbc, trim($_POST['sexo']));
    $cidade = mysqli_real_escape_string($dbc, trim($_POST['cidade']));
    $estado = mysqli_real_escape_string($dbc, trim($_POST['estado']));
    $foto_antiga = mysqli_real_escape_string($dbc, trim($_POST['foto_antiga']));
	
    $foto_nova = mysqli_real_escape_string($dbc, trim($_FILES['foto_nova']['name']));
    $foto_nova_type = $_FILES['foto_nova']['type'];
    $foto_nova_size = $_FILES['foto_nova']['size']; 
    list($foto_nova_width, $foto_nova_height) = getimagesize($_FILES['foto_nova']['tmp_name']);
    $error = false;

    // Validando os dados gravados nas variaveis
    if (!empty($foto_nova)) {
      if ((($foto_nova_type == 'image/gif') || ($foto_nova_type == 'image/jpeg') || ($foto_nova_type == 'image/pjpeg') ||
        ($foto_nova_type == 'image/png')) && ($foto_nova_size > 0) && ($foto_nova_size <= MAX_FILE_SIZE) &&
        ($foto_nova_width <= MAX_FILE_WIDTH) && ($foto_nova_height <= MAX_FILE_HEIGHT)) {
        if ($_FILES['foto_nova']['error'] == 0) {
          // Move o arquivo que esta na pasta temporaria para a pasta atual direcionada
          $target = UPLOAD_IMAGE . basename($foto_nova);
          if (move_uploaded_file($_FILES['foto_nova']['tmp_name'], $target)) {
            // A nova imagem foi salva e substituida pela imagem antiga
            if (!empty($foto_antiga) && ($foto_antiga != $foto_nova)) {
              @unlink(UPLOAD_IMAGE . $foto_antiga);
            }
          }
          else {
            // Se houver problema em carregar a imagem
            @unlink($_FILES['foto_nova']['tmp_name']);
            $error = true;
            echo '<p class="error">Desculpe, mais houve um problema em fazer upload da imagem.</p>';
          }
        }
      }
      else {
        // Se o arquivo, não for do formato desejado, exibir uma mensagem de erro!
        @unlink($_FILES['foto_nova']['tmp_name']);
        $error = true;
        echo '<p class="error">A foto tem que ser no formato GIF, JPEG, ou PNG e menor ou igual a ' . (MAX_FILE_SIZE / 1024) .
          ' KB e com ' . MAX_FILE_WIDTH . 'x' .MAX_FILE_HEIGHT . ' de tamanho.</p>';
      }
    }

    // Atualiza o seu perfil se não houver nenhum dado vazio
    if (!$error) {
      if (!empty($nome) && !empty($sobrenome) && !empty($sexo) && !empty($cidade) && !empty($estado)) {
        // Somente nova imagem 
        if (!empty($foto_nova)) {
          $query = "UPDATE redesocial_user SET data_user = NOW(), nome = '$nome', sobrenome = '$sobrenome', sexo = '$sexo', " .
            " cidade = '$cidade', estado = '$estado', foto = '$foto_nova' WHERE id = '" . $_SESSION['id'] . "'";
			$query2 = "UPDATE redesocial_status SET foto = '$foto_nova' WHERE user_id = '".$_SESSION['id']. "'";
			$query3 = "UPDATE redesocial_coment SET foto = '$foto_nova' WHERE user_id = '".$_SESSION['id']. "'";
        }
        else {
          $query = "UPDATE redesocial_user SET data_user = NOW(), nome = '$nome', sobrenome = '$sobrenome', sexo = '$sexo', " .
            " cidade = '$cidade', estado = '$estado' WHERE id = '" . $_SESSION['id'] . "'";
        }
		$data2 = mysqli_query($dbc, $query)or die(mysqli_error());
		$data3 = mysqli_query($dbc, $query2)or die(mysqli_error());
		$data4 = mysqli_query($dbc, $query3)or die(mysqli_error());
        // Mensagem de confirmação, se obter sucesso no envio dos dados
		echo '<br>';
		echo '<br>';
        echo '<p class="sucess">Seu perfil foi atualizado com sucesso.</p>';
		$home_url = 'http://'. $_SERVER['HTTP_HOST']. dirname($_SERVER['PHP_SELF']). '/index.php';
		header('Refresh: 2; '. $home_url);

        mysqli_close($dbc);
		require_once('footer.php');
        exit();
      }
      else {
        echo '<p class="error">Voce deve digitar todos os dados de seu perfil, a (imagem pode ser opcional).</p>';
      }
    }
  } // FIM
  else {
    // Consulta SQL para retornar os dados
    $query = "SELECT * FROM redesocial_user WHERE id = '" . $_SESSION['id'] . "'";
    $data = mysqli_query($dbc, $query);
    $row = mysqli_fetch_array($data);

    if ($row != NULL) {
      $nome = $row['nome'];
      $sobrenome = $row['sobrenome'];
      $sexo = $row['sexo'];
      $cidade = $row['cidade'];
      $estado = $row['estado'];
      $foto_antiga = $row['foto'];
    }
    else {
      echo '<p class="error">Desculpe, mais houve um problema ao tentar acessar o seu perfil.</p>';
    }
  }

  mysqli_close($dbc);
?>
<div class="form_settings">
  <form enctype="multipart/form-data" method="post" action="<?php echo $_SERVER['PHP_SELF']; ?>">
    <br>
    <br>
    <p class="sucess">Informações Pessoais</p>
    <br>
    <input type="hidden" name="MAX_FILE_SIZE" value="<?php echo MAX_FILE_SIZE; ?>" />
      <label for="nome" class="label">Nome: </label>
      <input type="text" name="nome" value="<?php if (!empty($nome)) echo $nome; ?>" /><br />
      <label for="sobrenome" class="label">Sobrenome: </label>
      <input type="text"  name="sobrenome" value="<?php if (!empty($sobrenome)) echo $sobrenome; ?>" /><br />
      <label for="sexo" class="label">Sexo:</label>
      <select name="sexo">
        <option value="M" <?php if (!empty($sexo) && $sexo == 'M') echo 'selected = "selected"'; ?>>Masculino</option>
        <option value="F" <?php if (!empty($sexo) && $sexo == 'F') echo 'selected = "selected"'; ?>>Feminino</option>
      </select><br />
      <label for="cidade" class="label">Cidade: </label>
      <input type="text" name="cidade" value="<?php if (!empty($cidade)) echo $cidade; ?>" /><br />
      <label for="estado" class="label">Estado: </label>
      
		<select id="estado" name="estado" >
		<option value="">Selecione</option>
		<option value="SP"<?php if (!empty($estado) && $estado == 'SP') echo 'selected = "selected"'; ?>>SP</option>
    	<option value="RJ"<?php if (!empty($estado) && $estado == 'RJ') echo 'selected = "selected"'; ?>>RJ</option>
    	<option value="ES"<?php if (!empty($estado) && $estado == 'ES') echo 'selected = "selected"'; ?>>ES</option>
    	<option value="MG"<?php if (!empty($estado) && $estado == 'MG') echo 'selected = "selected"'; ?>>MG</option>
    	<option value="RS"<?php if (!empty($estado) && $estado == 'RS') echo 'selected = "selected"'; ?>>RS</option>
    	<option value="SC"<?php if (!empty($estado) && $estado == 'SC') echo 'selected = "selected"'; ?>>SC</option> 
    	<option value="PR"<?php if (!empty($estado) && $estado == 'PR') echo 'selected = "selected"'; ?>>PR</option>
		</select><br />
        
      <input type="hidden" name="foto_antiga" value="<?php if (!empty($foto_antiga)) echo $foto_antiga; ?>" />
      <label for="foto_nova" class="label">Foto: </label>
      <input type="file" name="foto_nova" style="width:300"; />
      <?php if (!empty($foto_antiga)) {
        echo '<img class="profile" src="' . UPLOAD_IMAGE. $foto_antiga . '" alt="Foto de Perfil" width="80" height="80"/>';
      } ?>
      <br>
      <br>
    <input class="submit" type="submit" value="Salvar Perfil" name="submit" />
  </form>
  </div>
  
  <?php
require_once('footer.php');
?>
