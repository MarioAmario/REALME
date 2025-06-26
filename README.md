import React, { useState, useEffect } from 'react';
import { ShoppingCart, Plus, Minus, Trash2, Search, Package, FileText, Calculator } from 'lucide-react';

const PRODUTOS_KEY = 'pdv_produtos';
const VENDAS_KEY = 'pdv_vendas';
const CARRINHO_KEY = 'pdv_carrinho';

const defaultProdutos = [
  { id: 1, codigo: '7891000001', nome: 'Arroz Branco 5kg', preco: 25.90, categoria: 'Alimentos', estoque: 50 },
  { id: 2, codigo: '7891000002', nome: 'Feijão Carioca 1kg', preco: 8.50, categoria: 'Alimentos', estoque: 30 },
  { id: 3, codigo: '7891000003', nome: 'Óleo de Soja 900ml', preco: 5.75, categoria: 'Alimentos', estoque: 25 },
  { id: 4, codigo: '7891000004', nome: 'Açúcar Cristal 1kg', preco: 4.20, categoria: 'Alimentos', estoque: 40 },
  { id: 5, codigo: '7891000005', nome: 'Café Torrado 500g', preco: 12.90, categoria: 'Bebidas', estoque: 20 },
];

const getLocalData = (key, fallback) => {
  try {
    const raw = localStorage.getItem(key);
    if (!raw) return fallback;
    return JSON.parse(raw);
  } catch {
    return fallback;
  }
};

const SistemaPDV = () => {
  // Estados principais usando localStorage
  const [produtos, setProdutos] = useState(() => getLocalData(PRODUTOS_KEY, defaultProdutos));
  const [vendas, setVendas] = useState(() => getLocalData(VENDAS_KEY, []));
  const [carrinho, setCarrinho] = useState(() => getLocalData(CARRINHO_KEY, []));
  const [busca, setBusca] = useState('');
  const [codigoScanner, setCodigoScanner] = useState('');
  const [produtoSelecionado, setProdutoSelecionado] = useState(null);
  const [telaAtual, setTelaAtual] = useState('pdv');
  const [novoProduto, setNovoProduto] = useState({
    codigo: '',
    nome: '',
    preco: '',
    categoria: '',
    estoque: ''
  });
  const [produtoEditando, setProdutoEditando] = useState(null);
  const [aplicarDesconto, setAplicarDesconto] = useState(true);
  const [percentualDesconto, setPercentualDesconto] = useState(5);

  // Persistência no localStorage
  useEffect(() => {
    localStorage.setItem(PRODUTOS_KEY, JSON.stringify(produtos));
  }, [produtos]);
  useEffect(() => {
    localStorage.setItem(VENDAS_KEY, JSON.stringify(vendas));
  }, [vendas]);
  useEffect(() => {
    localStorage.setItem(CARRINHO_KEY, JSON.stringify(carrinho));
  }, [carrinho]);

  // Cálculos do carrinho
  const subtotal = carrinho.reduce((acc, item) => acc + (item.preco * item.quantidade), 0);
  const desconto = aplicarDesconto ? subtotal * (percentualDesconto / 100) : 0;
  const total = subtotal - desconto;

  // Função para scanner manual
  const processarCodigoScanner = (e) => {
    if (e.key === 'Enter') {
      const produto = produtos.find(p => p.codigo === codigoScanner);
      if (produto) {
        adicionarAoCarrinho(produto);
        setCodigoScanner('');
        setTimeout(() => {
          const scannerInput = document.getElementById('scanner-input');
          if (scannerInput) scannerInput.focus();
        }, 100);
      } else {
        alert('Produto não encontrado!');
        setCodigoScanner('');
      }
    }
  };

  // Função para editar produto
  const editarProduto = (produto) => {
    setProdutoEditando(produto);
  };

  const salvarEdicaoProduto = () => {
    setProdutos(produtos.map(p => 
      p.id === produtoEditando.id ? produtoEditando : p
    ));
    setProdutoEditando(null);
    alert('Produto atualizado com sucesso!');
  };

  const cancelarEdicao = () => {
    setProdutoEditando(null);
  };

  useEffect(() => {
    if (telaAtual === 'pdv') {
      setTimeout(() => {
        const scannerInput = document.getElementById('scanner-input');
        if (scannerInput) scannerInput.focus();
      }, 100);
    }
  }, [telaAtual]);

  const adicionarAoCarrinho = (produto) => {
    const itemExistente = carrinho.find(item => item.id === produto.id);
    if (itemExistente) {
      setCarrinho(carrinho.map(item =>
        item.id === produto.id
          ? { ...item, quantidade: item.quantidade + 1 }
          : item
      ));
    } else {
      setCarrinho([...carrinho, { ...produto, quantidade: 1 }]);
    }
  };

  const alterarQuantidade = (id, novaQuantidade) => {
    if (novaQuantidade <= 0) {
      removerDoCarrinho(id);
      return;
    }
    setCarrinho(carrinho.map(item =>
      item.id === id
        ? { ...item, quantidade: novaQuantidade }
        : item
    ));
  };

  const removerDoCarrinho = (id) => {
    setCarrinho(carrinho.filter(item => item.id !== id));
  };

  const finalizarVenda = () => {
    if (carrinho.length === 0) {
      alert('Carrinho vazio!');
      return;
    }

    const venda = {
      id: Date.now(),
      data: new Date().toLocaleString(),
      itens: [...carrinho],
      subtotal,
      desconto,
      total
    };

    setVendas([...vendas, venda]);
    setCarrinho([]);

    // Atualizar estoque
    setProdutos(produtos.map(produto => {
      const itemVendido = carrinho.find(item => item.id === produto.id);
      if (itemVendido) {
        return { ...produto, estoque: produto.estoque - itemVendido.quantidade };
      }
      return produto;
    }));

    alert('Venda finalizada com sucesso!');
  };

  const adicionarProduto = () => {
    if (!novoProduto.codigo || !novoProduto.nome || !novoProduto.preco) {
      alert('Preencha todos os campos obrigatórios!');
      return;
    }

    const produto = {
      id: Date.now(),
      codigo: novoProduto.codigo,
      nome: novoProduto.nome,
      preco: parseFloat(novoProduto.preco),
      categoria: novoProduto.categoria,
      estoque: parseInt(novoProduto.estoque)
    };

    setProdutos([...produtos, produto]);
    setNovoProduto({ codigo: '', nome: '', preco: '', categoria: '', estoque: '' });
    alert('Produto adicionado com sucesso!');
  };

  const produtosFiltrados = produtos.filter(produto =>
    produto.nome.toLowerCase().includes(busca.toLowerCase()) ||
    produto.codigo.includes(busca)
  );

  const TelaPDV = () => (
    <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
      {/* Lista de Produtos */}
      <div className="lg:col-span-2">
        <div className="bg-white rounded-lg shadow-lg p-6">
          {/* Scanner Manual */}
          <div className="mb-6 p-4 bg-blue-50 rounded-lg">
            <div className="flex items-center gap-2 mb-2">
              <div className="w-3 h-3 bg-green-500 rounded-full animate-pulse"></div>
              <h3 className="font-bold text-blue-800">Scanner Manual</h3>
            </div>
            <input
              id="scanner-input"
              type="text"
              placeholder="Digite ou escaneie o código do produto e pressione Enter..."
              value={codigoScanner}
              onChange={(e) => setCodigoScanner(e.target.value)}
              onKeyDown={processarCodigoScanner}
              className="w-full p-3 border-2 border-blue-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 text-lg font-mono"
              autoComplete="off"
              autoFocus
            />
            <p className="text-sm text-blue-600 mt-1">
              💡 Este campo mantém o foco para digitação contínua
            </p>
          </div>

          <div className="flex items-center gap-4 mb-4">
            <Search className="text-blue-600" size={20} />
            <input
              type="text"
              placeholder="Buscar produto por nome ou código..."
              value={busca}
              onChange={(e) => setBusca(e.target.value)}
              className="flex-1 p-3 border rounded-lg focus:ring-2 focus:ring-blue-500"
            />
          </div>
          
          <div className="max-h-96 overflow-y-auto">
            {produtosFiltrados.map(produto => (
              <div key={produto.id} className="flex items-center justify-between p-3 border-b hover:bg-gray-50">
                <div className="flex-1">
                  <h3 className="font-medium">{produto.nome}</h3>
                  <p className="text-sm text-gray-600">Código: {produto.codigo}</p>
                  <p className="text-sm text-gray-600">Estoque: {produto.estoque}</p>
                </div>
                <div className="text-right">
                  <p className="text-lg font-bold text-green-600">
                    R$ {produto.preco.toFixed(2)}
                  </p>
                  <button
                    onClick={() => {
                      adicionarAoCarrinho(produto);
                      setTimeout(() => {
                        const scannerInput = document.getElementById('scanner-input');
                        if (scannerInput) scannerInput.focus();
                      }, 100);
                    }}
                    className="mt-2 px-4 py-1 bg-blue-600 text-white rounded hover:bg-blue-700"
                    disabled={produto.estoque <= 0}
                  >
                    <Plus size={16} />
                  </button>
                </div>
              </div>
            ))}
          </div>
        </div>
      </div>

      {/* Carrinho de Compras */}
      <div className="bg-white rounded-lg shadow-lg p-6">
        <div className="flex items-center gap-2 mb-4">
          <ShoppingCart className="text-blue-600" size={20} />
          <h2 className="text-xl font-bold">Carrinho</h2>
        </div>

        <div className="max-h-64 overflow-y-auto mb-4">
          {carrinho.length === 0 ? (
            <p className="text-gray-500 text-center py-8">Carrinho vazio</p>
          ) : (
            carrinho.map(item => (
              <div key={item.id} className="flex items-center justify-between p-2 border-b">
                <div className="flex-1">
                  <h4 className="font-medium text-sm">{item.nome}</h4>
                  <p className="text-xs text-gray-600">R$ {item.preco.toFixed(2)}</p>
                </div>
                <div className="flex items-center gap-2">
                  <button
                    onClick={() => alterarQuantidade(item.id, item.quantidade - 1)}
                    className="p-1 bg-red-100 text-red-600 rounded hover:bg-red-200"
                  >
                    <Minus size={12} />
                  </button>
                  <span className="mx-2 font-medium">{item.quantidade}</span>
                  <button
                    onClick={() => alterarQuantidade(item.id, item.quantidade + 1)}
                    className="p-1 bg-green-100 text-green-600 rounded hover:bg-green-200"
                  >
                    <Plus size={12} />
                  </button>
                  <button
                    onClick={() => removerDoCarrinho(item.id)}
                    className="p-1 bg-gray-100 text-gray-600 rounded hover:bg-gray-200 ml-2"
                  >
                    <Trash2 size={12} />
                  </button>
                </div>
              </div>
            ))
          )}
        </div>

        {/* Totais */}
        <div className="border-t pt-4">
          {/* Controle de Desconto */}
          <div className="mb-4 p-3 bg-gray-50 rounded-lg">
            <div className="flex items-center justify-between mb-2">
              <label className="flex items-center gap-2">
                <input
                  type="checkbox"
                  checked={aplicarDesconto}
                  onChange={(e) => setAplicarDesconto(e.target.checked)}
                  className="w-4 h-4 text-blue-600"
                />
                <span className="font-medium">Aplicar Desconto</span>
              </label>
              {aplicarDesconto && (
                <div className="flex items-center gap-2">
                  <input
                    type="number"
                    value={percentualDesconto}
                    onChange={(e) => setPercentualDesconto(Math.max(0, Math.min(100, e.target.value)))}
                    className="w-16 p-1 border rounded text-center"
                    min="0"
                    max="100"
                  />
                  <span>%</span>
                </div>
              )}
            </div>
          </div>

          <div className="flex justify-between mb-2">
            <span>Subtotal:</span>
            <span>R$ {subtotal.toFixed(2)}</span>
          </div>
          {aplicarDesconto && (
            <div className="flex justify-between mb-2 text-green-600">
              <span>Desconto ({percentualDesconto}%):</span>
              <span>- R$ {desconto.toFixed(2)}</span>
            </div>
          )}
          <div className="flex justify-between text-xl font-bold border-t pt-2">
            <span>Total:</span>
            <span>R$ {total.toFixed(2)}</span>
          </div>
          
          <button
            onClick={() => {
              finalizarVenda();
              setTimeout(() => {
                const scannerInput = document.getElementById('scanner-input');
                if (scannerInput) scannerInput.focus();
              }, 100);
            }}
            className="w-full mt-4 p-3 bg-green-600 text-white rounded-lg hover:bg-green-700 font-medium"
            disabled={carrinho.length === 0}
          >
            Finalizar Venda
          </button>
        </div>
      </div>
    </div>
  );

  const TelaProdutos = () => (
    <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
      <div className="bg-white rounded-lg shadow-lg p-6">
        <div className="flex items-center gap-2 mb-4">
          <Package className="text-blue-600" size={20} />
          <h2 className="text-xl font-bold">Cadastrar Produto</h2>
        </div>
        
        <div className="space-y-4">
          <input
            type="text"
            placeholder="Código do produto"
            value={novoProduto.codigo}
            onChange={(e) => setNovoProduto({...novoProduto, codigo: e.target.value})}
            className="w-full p-3 border rounded-lg focus:ring-2 focus:ring-blue-500"
            autoComplete="off"
          />
          <input
            type="text"
            placeholder="Nome do produto"
            value={novoProduto.nome}
            onChange={(e) => setNovoProduto({...novoProduto, nome: e.target.value})}
            className="w-full p-3 border rounded-lg focus:ring-2 focus:ring-blue-500"
            autoComplete="off"
          />
          <input
            type="number"
            step="0.01"
            placeholder="Preço"
            value={novoProduto.preco}
            onChange={(e) => setNovoProduto({...novoProduto, preco: e.target.value})}
            className="w-full p-3 border rounded-lg focus:ring-2 focus:ring-blue-500"
            autoComplete="off"
          />
          <input
            type="text"
            placeholder="Categoria"
            value={novoProduto.categoria}
            onChange={(e) => setNovoProduto({...novoProduto, categoria: e.target.value})}
            className="w-full p-3 border rounded-lg focus:ring-2 focus:ring-blue-500"
            autoComplete="off"
          />
          <input
            type="number"
            placeholder="Estoque inicial"
            value={novoProduto.estoque}
            onChange={(e) => setNovoProduto({...novoProduto, estoque: e.target.value})}
            className="w-full p-3 border rounded-lg focus:ring-2 focus:ring-blue-500"
            autoComplete="off"
          />
          <button
            onClick={adicionarProduto}
            className="w-full p-3 bg-blue-600 text-white rounded-lg hover:bg-blue-700 font-medium"
          >
            Cadastrar Produto
          </button>
        </div>
      </div>

      <div className="bg-white rounded-lg shadow-lg p-6">
        <h2 className="text-xl font-bold mb-4">Lista de Produtos</h2>
        <div className="max-h-96 overflow-y-auto">
          {produtos.map(produto => (
            <div key={produto.id} className="p-3 border-b">
              {produtoEditando && produtoEditando.id === produto.id ? (
                <div className="space-y-3">
                  <div className="grid grid-cols-2 gap-2">
                    <input
                      type="text"
                      value={produtoEditando.codigo}
                      onChange={(e) => setProdutoEditando({...produtoEditando, codigo: e.target.value})}
                      className="p-2 border rounded focus:ring-2 focus:ring-blue-500"
                      placeholder="Código"
                    />
                    <input
                      type="text"
                      value={produtoEditando.nome}
                      onChange={(e) => setProdutoEditando({...produtoEditando, nome: e.target.value})}
                      className="p-2 border rounded focus:ring-2 focus:ring-blue-500"
                      placeholder="Nome"
                    />
                  </div>
                  <div className="grid grid-cols-3 gap-2">
                    <input
                      type="number"
                      step="0.01"
                      value={produtoEditando.preco}
                      onChange={(e) => setProdutoEditando({...produtoEditando, preco: parseFloat(e.target.value) || 0})}
                      className="p-2 border rounded focus:ring-2 focus:ring-blue-500"
                      placeholder="Preço"
                    />
                    <input
                      type="text"
                      value={produtoEditando.categoria}
                      onChange={(e) => setProdutoEditando({...produtoEditando, categoria: e.target.value})}
                      className="p-2 border rounded focus:ring-2 focus:ring-blue-500"
                      placeholder="Categoria"
                    />
                    <input
                      type="number"
                      value={produtoEditando.estoque}
                      onChange={(e) => setProdutoEditando({...produtoEditando, estoque: parseInt(e.target.value) || 0})}
                      className="p-2 border rounded focus:ring-2 focus:ring-blue-500"
                      placeholder="Estoque"
                    />
                  </div>
                  <div className="flex gap-2">
                    <button
                      onClick={salvarEdicaoProduto}
                      className="px-4 py-2 bg-green-600 text-white rounded hover:bg-green-700 text-sm"
                    >
                      Salvar
                    </button>
                    <button
                      onClick={cancelarEdicao}
                      className="px-4 py-2 bg-gray-600 text-white rounded hover:bg-gray-700 text-sm"
                    >
                      Cancelar
                    </button>
                  </div>
                </div>
              ) : (
                <div>
                  <div className="flex justify-between items-start mb-2">
                    <div className="flex-1">
                      <h3 className="font-medium">{produto.nome}</h3>
                      <p className="text-sm text-gray-600">Código: {produto.codigo}</p>
                      <p className="text-sm text-gray-600">Categoria: {produto.categoria}</p>
                    </div>
                    <button
                      onClick={() => editarProduto(produto)}
                      className="px-3 py-1 bg-blue-100 text-blue-600 rounded hover:bg-blue-200 text-sm"
                    >
                      Editar
                    </button>
                  </div>
                  <div className="flex justify-between items-center">
                    <span className="text-lg font-bold text-green-600">
                      R$ {produto.preco.toFixed(2)}
                    </span>
                    <span className={`px-2 py-1 rounded text-sm ${
                      produto.estoque > 10 ? 'bg-green-100 text-green-800' :
                      produto.estoque > 0 ? 'bg-yellow-100 text-yellow-800' :
                      'bg-red-100 text-red-800'
                    }`}>
                      Estoque: {produto.estoque}
                    </span>
                  </div>
                </div>
              )}
            </div>
          ))}
        </div>
      </div>
    </div>
  );

  const TelaRelatorios = () => (
    <div className="bg-white rounded-lg shadow-lg p-6">
      <div className="flex items-center gap-2 mb-4">
        <FileText className="text-blue-600" size={20} />
        <h2 className="text-xl font-bold">Relatório de Vendas</h2>
      </div>
      
      <div className="mb-6">
        <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
          <div className="bg-blue-50 p-4 rounded-lg">
            <h3 className="text-lg font-semibold text-blue-800">Total de Vendas</h3>
            <p className="text-2xl font-bold text-blue-600">{vendas.length}</p>
          </div>
          <div className="bg-green-50 p-4 rounded-lg">
            <h3 className="text-lg font-semibold text-green-800">Faturamento Total</h3>
            <p className="text-2xl font-bold text-green-600">
              R$ {vendas.reduce((acc, venda) => acc + venda.total, 0).toFixed(2)}
            </p>
          </div>
          <div className="bg-purple-50 p-4 rounded-lg">
            <h3 className="text-lg font-semibold text-purple-800">Ticket Médio</h3>
            <p className="text-2xl font-bold text-purple-600">
              R$ {vendas.length > 0 ? (vendas.reduce((acc, venda) => acc + venda.total, 0) / vendas.length).toFixed(2) : '0.00'}
            </p>
          </div>
        </div>
      </div>

      <div className="max-h-96 overflow-y-auto">
        {vendas.length === 0 ? (
          <p className="text-gray-500 text-center py-8">Nenhuma venda realizada</p>
        ) : (
          vendas.slice().reverse().map(venda => (
            <div key={venda.id} className="border-b p-4 hover:bg-gray-50">
              <div className="flex justify-between items-start mb-2">
                <h3 className="font-medium">Venda #{venda.id}</h3>
                <span className="text-sm text-gray-600">{venda.data}</span>
              </div>
              <div className="text-sm text-gray-600 mb-2">
                {venda.itens.map(item => (
                  <div key={item.id}>
                    {item.nome} - Qtd: {item.quantidade} - R$ {(item.preco * item.quantidade).toFixed(2)}
                  </div>
                ))}
              </div>
              <div className="text-right">
                <span className="text-lg font-bold text-green-600">
                  Total: R$ {venda.total.toFixed(2)}
                </span>
              </div>
            </div>
          ))
        )}
      </div>
    </div>
  );

  return (
    <div className="min-h-screen bg-gray-100">
      <header className="bg-white shadow-sm border-b">
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
          <div className="flex justify-between items-center py-4">
            <div className="flex items-center gap-2">
              <Calculator className="text-blue-600" size={24} />
              <h1 className="text-2xl font-bold text-gray-900">Sistema PDV</h1>
            </div>
            <nav className="flex gap-4">
              <button
                onClick={() => setTelaAtual('pdv')}
                className={`px-4 py-2 rounded-lg font-medium ${
                  telaAtual === 'pdv'
                    ? 'bg-blue-600 text-white'
                    : 'bg-gray-100 text-gray-700 hover:bg-gray-200'
                }`}
              >
                PDV
              </button>
              <button
                onClick={() => setTelaAtual('produtos')}
                className={`px-4 py-2 rounded-lg font-medium ${
                  telaAtual === 'produtos'
                    ? 'bg-blue-600 text-white'
                    : 'bg-gray-100 text-gray-700 hover:bg-gray-200'
                }`}
              >
                Produtos
              </button>
              <button
                onClick={() => setTelaAtual('relatorios')}
                className={`px-4 py-2 rounded-lg font-medium ${
                  telaAtual === 'relatorios'
                    ? 'bg-blue-600 text-white'
                    : 'bg-gray-100 text-gray-700 hover:bg-gray-200'
                }`}
              >
                Relatórios
              </button>
            </nav>
          </div>
        </div>
      </header>
      <main className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
        {telaAtual === 'pdv' && <TelaPDV />}
        {telaAtual === 'produtos' && <TelaProdutos />}
        {telaAtual === 'relatorios' && <TelaRelatorios />}
      </main>
    </div>
  );
};

export default SistemaPDV;
