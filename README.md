# trabalho01import hashlib
from flask_sqlalchemy import SQLAlchemy
db = SQLAlchemy()
class Usuario(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    nome = db.Column(db.String(100), nullable=False)
    email = db.Column(db.String(100), unique=True, nullable=False)
    senha_hash = db.Column(db.String(255), nullable=False)
    
    def __init__(self, nome, email, senha):
        self.nome = nome
        self.email = email
        self.senha_hash = self._gerar_hash_senha(senha)
    def _gerar_hash_senha(self, senha):
        return hashlib.sha256(senha.encode('utf-8')).hexdigest()
    
    def verificar_senha(self, senha):
        return self.senha_hash == self._gerar_hash_senha(senha)
    @classmethod
    def autenticar(cls, email, senha):
        usuario = cls.query.filter_by(email=email).first()
        if usuario and usuario.verificar_senha(senha):
            return usuario
        return None
class Produto(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    nome = db.Column(db.String(100), nullable=False)
    descricao = db.Column(db.String(255), nullable=True)
    preco = db.Column(db.Float, nullable=False)
    fornecedor_id = db.Column(db.Integer, db.ForeignKey('fornecedor.id'), nullable=False)
    fornecedor = db.relationship('Fornecedor', backref=db.backref('produtos', lazy=True))
    
    def __init__(self, nome, descricao, preco, fornecedor_id):
        self.nome = nome
        self.descricao = descricao
        self.preco = preco
        self.fornecedor_id = fornecedor_id
class Fornecedor(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    nome = db.Column(db.String(100), nullable=False)
    telefone = db.Column(db.String(20))
    endereco = db.Column(db.String(255))
    def __init__(self, nome, telefone, endereco):
        self.nome = nome
        self.telefone = telefone
        self.endereco = endereco
class Cesta(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    usuario_id = db.Column(db.Integer, db.ForeignKey('usuario.id'), nullable=False)
    usuario = db.relationship('Usuario', backref=db.backref('cestas', lazy=True))
    def __init__(self, usuario_id):
        self.usuario_id = usuario_id
    def adicionar_produto(self, produto):
        cesta_produto = CestaProduto(cesta_id=self.id, produto_id=produto.id)
        db.session.add(cesta_produto)
        db.session.commit()
    def calcular_total(self):
        total = 0
        for produto in self.produtos:
            total += produto.preco
        return total
    @property
    def produtos(self):
        return [produto for produto in Produto.query.filter(Produto.id.in_([cp.produto_id for cp in CestaProduto.query.filter_by(cesta_id=self.id)]))]
class CestaProduto(db.Model):
    cesta_id = db.Column(db.Integer, db.ForeignKey('cesta.id'), primary_key=True)
    produto_id = db.Column(db.Integer, db.ForeignKey('produto.id'), primary_key=True)
<form id="form_produto">
    <input type="text" id="produto_nome" placeholder="Nome do Produto">
    <input type="text" id="produto_descricao" placeholder="Descrição">
    <input type="number" id="produto_preco" placeholder="Preço">
    <select id="produto_fornecedor">
        <!-- Fornecedores serão carregados via AJAX -->
    </select>
    <button type="submit">Cadastrar Produto</button>
</form>
<script>
document.getElementById('form_produto').addEventListener('submit', function(event) {
    event.preventDefault();
    
    var nome = document.getElementById('produto_nome').value;
    var descricao = document.getElementById('produto_descricao').value;
    var preco = document.getElementById('produto_preco').value;
    var fornecedor_id = document.getElementById('produto_fornecedor').value;
    fetch('/cadastro-produto', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
            nome: nome,
            descricao: descricao,
            preco: preco,
            fornecedor_id: fornecedor_id
        })
    })
    .then(response => response.json())
    .then(data => {
        alert('Produto cadastrado com sucesso!');
        // Atualizar a lista de produtos via AJAX
    })
    .catch(error => console.error('Erro:', error));
});
</script>
