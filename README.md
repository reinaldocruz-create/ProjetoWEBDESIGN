<?php
// admin/index.php - Painel Administrativo Completo
session_start();

// Verificar se está logado e é admin
if(!isset($_SESSION['logado']) || $_SESSION['logado'] !== true || $_SESSION['usuario_tipo'] !== 'admin') {
    header('Location: ../index.php');
    exit;
}

include_once '../config/database.php';
include_once '../classes/Parceiro.php';
include_once '../classes/Candidato.php';
include_once '../classes/Contato.php';

$database = new Database();
$db = $database->getConnection();

// Buscar estatísticas
$parceiro = new Parceiro($db);
$candidato = new Candidato($db);
$contato = new Contato($db);

$parceirosPendentes = $parceiro->listarTodos('pendente');
$candidatosNovos = $candidato->listarTodos('novo');
$contatosNovos = $contato->listarTodos('novo');

$totalParceiros = count($parceiro->listarTodos());
$totalCandidatos = count($candidato->listarTodos());
$totalContatos = count($contato->listarTodos());
?>
<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Painel Administrativo - SoftTech</title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }

        :root {
            --laranja-500: #f97316;
            --laranja-600: #ea580c;
            --pedra-50: #fafaf9;
            --pedra-100: #f5f5f4;
            --pedra-200: #e7e5e4;
            --pedra-600: #57534e;
            --pedra-700: #44403c;
            --pedra-800: #292524;
            --pedra-900: #1c1917;
        }

        body {
            background-color: var(--pedra-100);
            color: var(--pedra-800);
        }

        .admin-header {
            background: linear-gradient(to right, var(--laranja-500), var(--laranja-600));
            color: white;
            padding: 20px 40px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
        }

        .admin-header-content {
            max-width: 1400px;
            margin: 0 auto;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .admin-title {
            font-size: 1.75rem;
            font-weight: 700;
        }

        .admin-user {
            display: flex;
            align-items: center;
            gap: 16px;
        }

        .btn {
            padding: 10px 20px;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            font-weight: 600;
            text-decoration: none;
            display: inline-block;
            transition: all 0.3s ease;
        }

        .btn-secondary {
            background: white;
            color: var(--laranja-600);
        }

        .btn-secondary:hover {
            background: var(--pedra-100);
        }

        .admin-container {
            max-width: 1400px;
            margin: 40px auto;
            padding: 0 40px;
        }

        .stats-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
            gap: 24px;
            margin-bottom: 40px;
        }

        .stat-card {
            background: white;
            padding: 24px;
            border-radius: 16px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.05);
            display: flex;
            justify-content: space-between;
            align-items: center;
            transition: transform 0.3s ease;
        }

        .stat-card:hover {
            transform: translateY(-4px);
            box-shadow: 0 8px 12px rgba(0,0,0,0.1);
        }

        .stat-info h3 {
            color: var(--pedra-600);
            font-size: 0.875rem;
            margin-bottom: 8px;
        }

        .stat-info .stat-number {
            font-size: 2.5rem;
            font-weight: 700;
            color: var(--pedra-900);
        }

        .stat-icon {
            width: 64px;
            height: 64px;
            border-radius: 12px;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 1.75rem;
        }

        .icon-orange {
            background: linear-gradient(135deg, var(--laranja-500), var(--laranja-600));
            color: white;
        }

        .icon-blue {
            background: linear-gradient(135deg, #3b82f6, #2563eb);
            color: white;
        }

        .icon-green {
            background: linear-gradient(135deg, #10b981, #059669);
            color: white;
        }

        .section-card {
            background: white;
            border-radius: 16px;
            padding: 32px;
            margin-bottom: 32px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.05);
        }

        .section-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 24px;
            padding-bottom: 16px;
            border-bottom: 2px solid var(--pedra-100);
        }

        .section-title {
            font-size: 1.5rem;
            font-weight: 700;
            color: var(--pedra-900);
        }

        .badge {
            background: var(--laranja-500);
            color: white;
            padding: 4px 12px;
            border-radius: 12px;
            font-size: 0.875rem;
            font-weight: 600;
        }

        .table {
            width: 100%;
            border-collapse: collapse;
        }

        .table th {
            text-align: left;
            padding: 12px;
            background: var(--pedra-50);
            font-weight: 600;
            color: var(--pedra-700);
            font-size: 0.875rem;
            text-transform: uppercase;
        }

        .table td {
            padding: 16px 12px;
            border-bottom: 1px solid var(--pedra-100);
        }

        .table tr:hover {
            background: var(--pedra-50);
        }

        .status {
            padding: 4px 12px;
            border-radius: 12px;
            font-size: 0.75rem;
            font-weight: 600;
            text-transform: uppercase;
        }

        .status-pendente {
            background: #fef3c7;
            color: #92400e;
        }

        .status-novo {
            background: #dbeafe;
            color: #1e40af;
        }

        .status-aprovado {
            background: #d1fae5;
            color: #065f46;
        }

        .btn-action {
            padding: 6px 12px;
            font-size: 0.875rem;
            margin-right: 8px;
        }

        .btn-success {
            background: #10b981;
            color: white;
        }

        .btn-success:hover {
            background: #059669;
        }

        .btn-danger {
            background: #ef4444;
            color: white;
        }

        .btn-danger:hover {
            background: #dc2626;
        }

        .btn-info {
            background: #3b82f6;
            color: white;
        }

        .btn-info:hover {
            background: #2563eb;
        }

        .empty-state {
            text-align: center;
            padding: 40px;
            color: var(--pedra-600);
        }

        .empty-state i {
            font-size: 3rem;
            margin-bottom: 16px;
            opacity: 0.3;
        }

        .modal-overlay {
            display: none;
            position: fixed;
            inset: 0;
            background: rgba(0,0,0,0.5);
            z-index: 1000;
            align-items: center;
            justify-content: center;
            padding: 20px;
        }

        .modal-overlay.active {
            display: flex;
        }

        .modal-content {
            background: white;
            border-radius: 16px;
            max-width: 600px;
            width: 100%;
            padding: 32px;
            position: relative;
            max-height: 80vh;
            overflow-y: auto;
        }

        .modal-close {
            position: absolute;
            top: 16px;
            right: 16px;
            background: none;
            border: none;
            font-size: 24px;
            cursor: pointer;
            color: var(--pedra-600);
        }

        .modal-close:hover {
            color: var(--pedra-900);
        }

        .modal-title {
            font-size: 1.5rem;
            font-weight: 700;
            margin-bottom: 16px;
            color: var(--pedra-900);
        }

        .modal-text {
            background: var(--pedra-50);
            padding: 16px;
            border-radius: 8px;
            line-height: 1.6;
            color: var(--pedra-700);
        }

        @media (max-width: 768px) {
            .admin-container {
                padding: 0 20px;
            }

            .stats-grid {
                grid-template-columns: 1fr;
            }

            .table {
                font-size: 0.875rem;
            }

            .section-header {
                flex-direction: column;
                align-items: flex-start;
                gap: 12px;
            }
        }
    </style>
</head>
<body>
    <header class="admin-header">
        <div class="admin-header-content">
            <div>
                <h1 class="admin-title">
                    <i class="fas fa-user-shield"></i> Painel Administrativo
                </h1>
            </div>
            <div class="admin-user">
                <span>Olá, <?php echo htmlspecialchars($_SESSION['usuario_nome']); ?>!</span>
                <a href="../index.php" class="btn btn-secondary">
                    <i class="fas fa-home"></i> Voltar ao Site
                </a>
            </div>
        </div>
    </header>

    <div class="admin-container">
        <!-- Estatísticas -->
        <div class="stats-grid">
            <div class="stat-card">
                <div class="stat-info">
                    <h3>Total de Parceiros</h3>
                    <div class="stat-number"><?php echo $totalParceiros; ?></div>
                </div>
                <div class="stat-icon icon-orange">
                    <i class="fas fa-handshake"></i>
                </div>
            </div>

            <div class="stat-card">
                <div class="stat-info">
                    <h3>Total de Candidatos</h3>
                    <div class="stat-number"><?php echo $totalCandidatos; ?></div>
                </div>
                <div class="stat-icon icon-blue">
                    <i class="fas fa-users"></i>
                </div>
            </div>

            <div class="stat-card">
                <div class="stat-info">
                    <h3>Total de Contatos</h3>
                    <div class="stat-number"><?php echo $totalContatos; ?></div>
                </div>
                <div class="stat-icon icon-green">
                    <i class="fas fa-envelope"></i>
                </div>
            </div>
        </div>

        <!-- Parceiros Pendentes -->
        <div class="section-card">
            <div class="section-header">
                <h2 class="section-title">
                    <i class="fas fa-handshake"></i> Solicitações de Parceria
                </h2>
                <span class="badge"><?php echo count($parceirosPendentes); ?> Pendentes</span>
            </div>
            
            <?php if(count($parceirosPendentes) > 0): ?>
            <div style="overflow-x: auto;">
                <table class="table">
                    <thead>
                        <tr>
                            <th>Empresa</th>
                            <th>CNPJ</th>
                            <th>Segmento</th>
                            <th>Contato</th>
                            <th>Data</th>
                            <th>Status</th>
                            <th>Ações</th>
                        </tr>
                    </thead>
                    <tbody>
                        <?php foreach($parceirosPendentes as $p): ?>
                        <tr>
                            <td><strong><?php echo htmlspecialchars($p['nome_empresa']); ?></strong></td>
                            <td><?php echo htmlspecialchars($p['cnpj'] ?? '-'); ?></td>
                            <td><?php echo ucfirst($p['segmento']); ?></td>
                            <td>
                                <?php echo htmlspecialchars($p['nome']); ?><br>
                                <small><?php echo htmlspecialchars($p['email']); ?></small>
                            </td>
                            <td><?php echo date('d/m/Y', strtotime($p['data_solicitacao'])); ?></td>
                            <td><span class="status status-pendente"><?php echo $p['status']; ?></span></td>
                            <td>
                                <button class="btn btn-action btn-success" onclick="aprovarParceiro(<?php echo $p['id']; ?>)">
                                    <i class="fas fa-check"></i> Aprovar
                                </button>
                                <button class="btn btn-action btn-danger" onclick="rejeitarParceiro(<?php echo $p['id']; ?>)">
                                    <i class="fas fa-times"></i> Rejeitar
                                </button>
                            </td>
                        </tr>
                        <?php endforeach; ?>
                    </tbody>
                </table>
            </div>
            <?php else: ?>
            <div class="empty-state">
                <i class="fas fa-inbox"></i>
                <p>Nenhuma solicitação de parceria pendente</p>
            </div>
            <?php endif; ?>
        </div>

        <!-- Candidatos Novos -->
        <div class="section-card">
            <div class="section-header">
                <h2 class="section-title">
                    <i class="fas fa-briefcase"></i> Novas Candidaturas
                </h2>
                <span class="badge"><?php echo count($candidatosNovos); ?> Novos</span>
            </div>
            
            <?php if(count($candidatosNovos) > 0): ?>
            <div style="overflow-x: auto;">
                <table class="table">
                    <thead>
                        <tr>
                            <th>Nome</th>
                            <th>Cargo</th>
                            <th>Experiência</th>
                            <th>Contato</th>
                            <th>Data</th>
                            <th>Status</th>
                            <th>Ações</th>
                        </tr>
                    </thead>
                    <tbody>
                        <?php foreach($candidatosNovos as $c): ?>
                        <tr>
                            <td><strong><?php echo htmlspecialchars($c['nome_completo']); ?></strong></td>
                            <td><?php echo htmlspecialchars($c['cargo_interesse']); ?></td>
                            <td><?php echo ucfirst($c['nivel_experiencia'] ?? '-'); ?></td>
                            <td>
                                <?php echo htmlspecialchars($c['email']); ?><br>
                                <small><?php echo htmlspecialchars($c['telefone'] ?? '-'); ?></small>
                            </td>
                            <td><?php echo date('d/m/Y', strtotime($c['data_candidatura'])); ?></td>
                            <td><span class="status status-novo"><?php echo $c['status']; ?></span></td>
                            <td>
                                <?php if($c['curriculo_arquivo']): ?>
                                <a href="../<?php echo $c['curriculo_arquivo']; ?>" class="btn btn-action btn-info" target="_blank">
                                    <i class="fas fa-file-pdf"></i> Currículo
                                </a>
                                <?php endif; ?>
                                <button class="btn btn-action btn-success" onclick="analisarCandidato(<?php echo $c['id']; ?>)">
                                    <i class="fas fa-eye"></i> Analisar
                                </button>
                            </td>
                        </tr>
                        <?php endforeach; ?>
                    </tbody>
                </table>
            </div>
            <?php else: ?>
            <div class="empty-state">
                <i class="fas fa-inbox"></i>
                <p>Nenhuma candidatura nova</p>
            </div>
            <?php endif; ?>
        </div>

        <!-- Contatos Novos -->
        <div class="section-card">
            <div class="section-header">
                <h2 class="section-title">
                    <i class="fas fa-envelope"></i> Mensagens Recebidas
                </h2>
                <span class="badge"><?php echo count($contatosNovos); ?> Novas</span>
            </div>
            
            <?php if(count($contatosNovos) > 0): ?>
            <div style="overflow-x: auto;">
                <table class="table">
                    <thead>
                        <tr>
                            <th>Nome</th>
                            <th>Email</th>
                            <th>Assunto</th>
                            <th>Tipo</th>
                            <th>Data</th>
                            <th>Ações</th>
                        </tr>
                    </thead>
                    <tbody>
                        <?php foreach($contatosNovos as $ct): ?>
                        <tr>
                            <td><strong><?php echo htmlspecialchars($ct['nome']); ?></strong></td>
                            <td><?php echo htmlspecialchars($ct['email']); ?></td>
                            <td><?php echo htmlspecialchars($ct['assunto']); ?></td>
                            <td><?php echo ucfirst($ct['tipo']); ?></td>
                            <td><?php echo date('d/m/Y H:i', strtotime($ct['data_envio'])); ?></td>
                            <td>
                                <button class="btn btn-action btn-info" onclick="verMensagem(<?php echo $ct['id']; ?>, '<?php echo htmlspecialchars($ct['mensagem'], ENT_QUOTES); ?>')">
                                    <i class="fas fa-eye"></i> Ver
                                </button>
                                <button class="btn btn-action btn-success" onclick="marcarRespondido(<?php echo $ct['id']; ?>)">
                                    <i class="fas fa-check"></i> Responder
                                </button>
                            </td>
                        </tr>
                        <?php endforeach; ?>
                    </tbody>
                </table>
            </div>
            <?php else: ?>
            <div class="empty-state">
                <i class="fas fa-inbox"></i>
                <p>Nenhuma mensagem nova</p>
            </div>
            <?php endif; ?>
        </div>
    </div>

    <!-- Modal Mensagem -->
    <div id="modal-mensagem" class="modal-overlay">
        <div class="modal-content">
            <button class="modal-close" onclick="fecharModalMensagem()">&times;</button>
            <h3 class="modal-title">Mensagem</h3>
            <div id="conteudo-mensagem" class="modal-text"></div>
        </div>
    </div>

    <script>
        function aprovarParceiro(id) {
            if(confirm('Deseja aprovar esta solicitação de parceria?')) {
                fetch('../api/admin/aprovar-parceiro.php', {
                    method: 'POST',
                    headers: {'Content-Type': 'application/json'},
                    body: JSON.stringify({id: id})
                })
                .then(response => response.json())
                .then(data => {
                    if(data.success) {
                        alert('Parceiro aprovado com sucesso!');
                        location.reload();
                    } else {
                        alert('Erro: ' + data.message);
                    }
                })
                .catch(error => {
                    console.error('Erro:', error);
                    alert('Erro ao aprovar parceiro');
                });
            }
        }

        function rejeitarParceiro(id) {
            if(confirm('Deseja rejeitar esta solicitação de parceria?')) {
                fetch('../api/admin/rejeitar-parceiro.php', {
                    method: 'POST',
                    headers: {'Content-Type': 'application/json'},
                    body: JSON.stringify({id: id})
                })
                .then(response => response.json())
                .then(data => {
                    if(data.success) {
                        alert('Parceiro rejeitado');
                        location.reload();
                    } else {
                        alert('Erro: ' + data.message);
                    }
                })
                .catch(error => {
                    console.error('Erro:', error);
                    alert('Erro ao rejeitar parceiro');
                });
            }
        }

        function analisarCandidato(id) {
            if(confirm('Marcar candidato como "Em Análise"?')) {
                fetch('../api/admin/analisar-candidato.php', {
                    method: 'POST',
                    headers: {'Content-Type': 'application/json'},
                    body: JSON.stringify({id: id})
                })
                .then(response => response.json())
                .then(data => {
                    if(data.success) {
                        alert('Status atualizado com sucesso!');
                        location.reload();
                    } else {
                        alert('Erro: ' + data.message);
                    }
                })
                .catch(error => {
                    console.error('Erro:', error);
                    alert('Erro ao atualizar status');
                });
            }
        }

        function verMensagem(id, mensagem) {
            document.getElementById('conteudo-mensagem').textContent = mensagem;
            document.getElementById('modal-mensagem').classList.add('active');
        }

        function fecharModalMensagem() {
            document.getElementById('modal-mensagem').classList.remove('active');
        }

        function marcarRespondido(id) {
            if(confirm('Marcar esta mensagem como respondida?')) {
                fetch('../api/admin/marcar-respondido.php', {
                    method: 'POST',
                    headers: {'Content-Type': 'application/json'},
                    body: JSON.stringify({id: id})
                })
                .then(response => response.json())
                .then(data => {
                    if(data.success) {
                        alert('Mensagem marcada como respondida!');
                        location.reload();
                    } else {
                        alert('Erro: ' + data.message);
                    }
                })
                .catch(error => {
                    console.error('Erro:', error);
                    alert('Erro ao atualizar status');
                });
            }
        }

        // Fechar modal ao clicar fora
        document.getElementById('modal-mensagem').addEventListener('click', function(e) {
            if(e.target === this) {
                fecharModalMensagem();
            }
        });
    </script>
</body>
</html>
