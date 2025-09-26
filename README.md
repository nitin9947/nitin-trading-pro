# nitin-trading-pro
ths website for demo trading
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>TradeAI Pro - Advanced Finance & Trading Platform</title>
    <script src="https://unpkg.com/react@18/umd/react.development.js"></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body { font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif; }
        .ticker-scroll { animation: scroll 25s linear infinite; }
        @keyframes scroll { 0% { transform: translateX(100%); } 100% { transform: translateX(-100%); } }
        .glow-card { box-shadow: 0 4px 20px rgba(59, 130, 246, 0.15); }
        .price-up { animation: flashGreen 0.5s ease-out; }
        .price-down { animation: flashRed 0.5s ease-out; }
        @keyframes flashGreen { 0% { background-color: rgba(34, 197, 94, 0.2); } 100% { background-color: transparent; } }
        @keyframes flashRed { 0% { background-color: rgba(239, 68, 68, 0.2); } 100% { background-color: transparent; } }
        .gradient-text { background: linear-gradient(135deg, #3b82f6, #8b5cf6); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }
    </style>
</head>
<body class="bg-slate-900 text-white">
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useEffect, useRef } = React;

        // Icon Components (compressed)
        const icons = {
            TrendingUp: ({size=20, className=""}) => <svg className={className} width={size} height={size} fill="none" stroke="currentColor" strokeWidth="2" viewBox="0 0 24 24"><polyline points="22,7 13.5,15.5 8.5,10.5 2,17"/><polyline points="16,7 22,7 22,13"/></svg>,
            TrendingDown: ({size=20, className=""}) => <svg className={className} width={size} height={size} fill="none" stroke="currentColor" strokeWidth="2" viewBox="0 0 24 24"><polyline points="22,17 13.5,8.5 8.5,13.5 2,7"/><polyline points="16,17 22,17 22,11"/></svg>,
            Activity: ({size=20, className=""}) => <svg className={className} width={size} height={size} fill="none" stroke="currentColor" strokeWidth="2" viewBox="0 0 24 24"><path d="M22 12h-4l-3 9L9 3l-3 9H2"/></svg>,
            DollarSign: ({size=20, className=""}) => <svg className={className} width={size} height={size} fill="none" stroke="currentColor" strokeWidth="2" viewBox="0 0 24 24"><line x1="12" x2="12" y1="2" y2="22"/><path d="M17 5H9.5a3.5 3.5 0 0 0 0 7h5a3.5 3.5 0 0 1 0 7H6"/></svg>,
            BarChart: ({size=20, className=""}) => <svg className={className} width={size} height={size} fill="none" stroke="currentColor" strokeWidth="2" viewBox="0 0 24 24"><path d="M3 3v18h18"/><path d="M18 17V9"/><path d="M13 17V5"/><path d="M8 17v-3"/></svg>,
            Brain: ({size=20, className=""}) => <svg className={className} width={size} height={size} fill="none" stroke="currentColor" strokeWidth="2" viewBox="0 0 24 24"><path d="M12 5a3 3 0 1 0-5.997.125 4 4 0 0 0-2.526 5.77 4 4 0 0 0 .556 6.588A4 4 0 1 0 12 18Z"/><path d="M12 5a3 3 0 1 1 5.997.125 4 4 0 0 1 2.526 5.77 4 4 0 0 1-.556 6.588A4 4 0 1 1 12 18Z"/></svg>,
            Bell: ({size=20, className=""}) => <svg className={className} width={size} height={size} fill="none" stroke="currentColor" strokeWidth="2" viewBox="0 0 24 24"><path d="M6 8a6 6 0 0 1 12 0c0 7 3 9 3 9H3s3-2 3-9"/><path d="m13.73 21a2 2 0 0 1-3.46 0"/></svg>,
            Plus: ({size=20, className=""}) => <svg className={className} width={size} height={size} fill="none" stroke="currentColor" strokeWidth="2" viewBox="0 0 24 24"><path d="M5 12h14"/><path d="M12 5v14"/></svg>,
            Search: ({size=20, className=""}) => <svg className={className} width={size} height={size} fill="none" stroke="currentColor" strokeWidth="2" viewBox="0 0 24 24"><circle cx="11" cy="11" r="8"/><path d="m21 21-4.35-4.35"/></svg>,
            Target: ({size=20, className=""}) => <svg className={className} width={size} height={size} fill="none" stroke="currentColor" strokeWidth="2" viewBox="0 0 24 24"><circle cx="12" cy="12" r="10"/><circle cx="12" cy="12" r="6"/><circle cx="12" cy="12" r="2"/></svg>,
            CreditCard: ({size=20, className=""}) => <svg className={className} width={size} height={size} fill="none" stroke="currentColor" strokeWidth="2" viewBox="0 0 24 24"><rect width="20" height="14" x="2" y="5" rx="2"/><path d="M2 10h20"/></svg>
        };

        const TradeAIPro = () => {
            // Main state
            const [activeTab, setActiveTab] = useState('dashboard');
            const [portfolioValue, setPortfolioValue] = useState(157420.50);
            const [buyingPower, setBuyingPower] = useState(25000);
            const [showTradeModal, setShowTradeModal] = useState(false);
            const [selectedStock, setSelectedStock] = useState(null);
            const [tradeType, setTradeType] = useState('buy');
            const [tradeQuantity, setTradeQuantity] = useState(1);
            const [showAIChat, setShowAIChat] = useState(false);
            const [chatMessage, setChatMessage] = useState('');
            const [conversation, setConversation] = useState([]);
            const chartRef = useRef(null);
            const chartInstance = useRef(null);

            // Market data
            const [stockData, setStockData] = useState({
                'AAPL': { symbol: 'AAPL', name: 'Apple Inc.', price: 185.92, change: 2.43, changePercent: 1.33, volume: '52.3M', history: [] },
                'MSFT': { symbol: 'MSFT', name: 'Microsoft Corp.', price: 378.85, change: -1.25, changePercent: -0.33, volume: '28.1M', history: [] },
                'GOOGL': { symbol: 'GOOGL', name: 'Alphabet Inc.', price: 138.21, change: 0.87, changePercent: 0.63, volume: '31.5M', history: [] },
                'AMZN': { symbol: 'AMZN', name: 'Amazon.com Inc.', price: 144.32, change: 3.21, changePercent: 2.28, volume: '45.7M', history: [] },
                'TSLA': { symbol: 'TSLA', name: 'Tesla Inc.', price: 248.50, change: -5.67, changePercent: -2.23, volume: '78.9M', history: [] },
                'NVDA': { symbol: 'NVDA', name: 'NVIDIA Corp.', price: 465.20, change: 12.45, changePercent: 2.75, volume: '62.4M', history: [] },
                'META': { symbol: 'META', name: 'Meta Platforms', price: 298.58, change: 4.32, changePercent: 1.47, volume: '34.2M', history: [] },
                'SPY': { symbol: 'SPY', name: 'SPDR S&P 500', price: 445.23, change: 1.87, changePercent: 0.42, volume: '89.1M', history: [] }
            });

            const [positions, setPositions] = useState([
                { symbol: 'AAPL', shares: 50, avgPrice: 175.30, currentPrice: 185.92, value: 9296 },
                { symbol: 'MSFT', shares: 25, avgPrice: 365.80, currentPrice: 378.85, value: 9471.25 },
                { symbol: 'NVDA', shares: 30, avgPrice: 420.60, currentPrice: 465.20, value: 13956 },
                { symbol: 'TSLA', shares: 40, avgPrice: 255.75, currentPrice: 248.50, value: 9940 }
            ]);

            const [watchlist, setWatchlist] = useState(['GOOGL', 'AMZN', 'META', 'SPY']);
            const [orders, setOrders] = useState([]);
            const [news, setNews] = useState([
                { title: 'Apple announces new AI features for iOS', time: '2h ago', impact: 'positive' },
                { title: 'Microsoft beats Q3 earnings expectations', time: '4h ago', impact: 'positive' },
                { title: 'Tesla recall affects 100,000 vehicles', time: '6h ago', impact: 'negative' },
                { title: 'NVIDIA launches new AI chip architecture', time: '8h ago', impact: 'positive' }
            ]);

            // Initialize price history
            useEffect(() => {
                const generateHistory = (price) => {
                    const history = [];
                    let basePrice = price * 0.98;
                    for (let i = 0; i < 50; i++) {
                        basePrice *= (1 + (Math.random() - 0.5) * 0.02);
                        history.push({ time: new Date(Date.now() - (49-i) * 60000), price: parseFloat(basePrice.toFixed(2)) });
                    }
                    return history;
                };

                setStockData(prev => {
                    const newData = { ...prev };
                    Object.keys(newData).forEach(symbol => {
                        newData[symbol].history = generateHistory(newData[symbol].price);
                    });
                    return newData;
                });
            }, []);

            // Real-time updates
            useEffect(() => {
                const interval = setInterval(() => {
                    setStockData(prev => {
                        const newData = { ...prev };
                        Object.keys(newData).forEach(symbol => {
                            const changePercent = (Math.random() - 0.5) * 0.008;
                            const newPrice = newData[symbol].price * (1 + changePercent);
                            const priceChange = newPrice - newData[symbol].price;
                            
                            newData[symbol] = {
                                ...newData[symbol],
                                price: parseFloat(newPrice.toFixed(2)),
                                change: parseFloat(priceChange.toFixed(2)),
                                changePercent: parseFloat((changePercent * 100).toFixed(2)),
                                history: [...newData[symbol].history.slice(1), {
                                    time: new Date(),
                                    price: parseFloat(newPrice.toFixed(2))
                                }]
                            };
                        });
                        return newData;
                    });

                    // Update positions
                    setPositions(prev => prev.map(pos => ({
                        ...pos,
                        currentPrice: stockData[pos.symbol]?.price || pos.currentPrice,
                        value: pos.shares * (stockData[pos.symbol]?.price || pos.currentPrice)
                    })));
                }, 3000);

                return () => clearInterval(interval);
            }, [stockData]);

            // Chart rendering
            useEffect(() => {
                if (selectedStock && chartRef.current && stockData[selectedStock]) {
                    const ctx = chartRef.current.getContext('2d');
                    if (chartInstance.current) chartInstance.current.destroy();
                    
                    const data = stockData[selectedStock];
                    chartInstance.current = new Chart(ctx, {
                        type: 'line',
                        data: {
                            labels: data.history.map(p => p.time.toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'})),
                            datasets: [{
                                data: data.history.map(p => p.price),
                                borderColor: data.change >= 0 ? '#22c55e' : '#ef4444',
                                backgroundColor: data.change >= 0 ? '#22c55e20' : '#ef444420',
                                fill: true,
                                tension: 0.4,
                                pointRadius: 0,
                                borderWidth: 2
                            }]
                        },
                        options: {
                            responsive: true,
                            maintainAspectRatio: false,
                            scales: {
                                x: { display: false },
                                y: { 
                                    display: true,
                                    grid: { color: '#374151' },
                                    ticks: { color: '#9ca3af', callback: value => '$' + value.toFixed(2) }
                                }
                            },
                            plugins: { legend: { display: false } }
                        }
                    });
                }
            }, [selectedStock, stockData]);

            // Trading functions
            const executeTrade = () => {
                if (!selectedStock || tradeQuantity <= 0) return;
                
                const stock = stockData[selectedStock];
                const totalCost = stock.price * tradeQuantity;
                
                if (tradeType === 'buy' && totalCost > buyingPower) {
                    alert('Insufficient buying power!');
                    return;
                }

                const newOrder = {
                    id: Date.now(),
                    symbol: selectedStock,
                    type: tradeType,
                    quantity: tradeQuantity,
                    price: stock.price,
                    timestamp: new Date(),
                    status: 'filled'
                };

                setOrders(prev => [newOrder, ...prev]);

                if (tradeType === 'buy') {
                    setBuyingPower(prev => prev - totalCost);
                    setPositions(prev => {
                        const existing = prev.find(p => p.symbol === selectedStock);
                        if (existing) {
                            const newShares = existing.shares + tradeQuantity;
                            const newAvgPrice = ((existing.shares * existing.avgPrice) + totalCost) / newShares;
                            return prev.map(p => 
                                p.symbol === selectedStock 
                                    ? { ...p, shares: newShares, avgPrice: newAvgPrice, value: newShares * stock.price }
                                    : p
                            );
                        } else {
                            return [...prev, {
                                symbol: selectedStock,
                                shares: tradeQuantity,
                                avgPrice: stock.price,
                                currentPrice: stock.price,
                                value: totalCost
                            }];
                        }
                    });
                } else {
                    setBuyingPower(prev => prev + totalCost);
                    setPositions(prev => prev.map(p => 
                        p.symbol === selectedStock 
                            ? { ...p, shares: Math.max(0, p.shares - tradeQuantity), value: Math.max(0, p.shares - tradeQuantity) * stock.price }
                            : p
                    ).filter(p => p.shares > 0));
                }

                setShowTradeModal(false);
                setTradeQuantity(1);
            };

            // AI Chat
            const handleChatSubmit = (e) => {
                e.preventDefault();
                if (!chatMessage.trim()) return;

                const userMessage = { role: 'user', content: chatMessage };
                setConversation(prev => [...prev, userMessage]);
                
                setTimeout(() => {
                    let response = '';
                    const msg = chatMessage.toLowerCase();
                    
                    if (msg.includes('buy') || msg.includes('invest')) {
                        response = "Based on current market analysis, I recommend diversified positions in tech stocks. NVDA shows strong momentum with AI developments.";
                    } else if (msg.includes('sell') || msg.includes('profit')) {
                        response = "Consider taking profits on positions up >20%. Your NVDA position shows strong gains - might be worth scaling out partially.";
                    } else if (msg.includes('market') || msg.includes('trend')) {
                        response = "Current market shows bullish sentiment. Tech sector leading gains. Watch for Fed announcements next week for volatility.";
                    } else {
                        response = "I'm analyzing your portfolio. You have strong positions in growth stocks. Consider adding some defensive plays for balance.";
                    }
                    
                    setConversation(prev => [...prev, { role: 'assistant', content: response }]);
                }, 1000);
                
                setChatMessage('');
            };

            const totalGainLoss = positions.reduce((acc, pos) => acc + (pos.currentPrice - pos.avgPrice) * pos.shares, 0);
            const totalGainLossPercent = positions.length > 0 ? (totalGainLoss / positions.reduce((acc, pos) => acc + pos.avgPrice * pos.shares, 0)) * 100 : 0;

            return (
                <div className="min-h-screen bg-slate-900">
                    {/* Market Ticker */}
                    <div className="bg-black bg-opacity-50 py-2 overflow-hidden border-b border-slate-700">
                        <div className="ticker-scroll flex space-x-8 text-sm">
                            {Object.values(stockData).map((stock, idx) => (
                                <div key={idx} className="flex items-center space-x-2 whitespace-nowrap">
                                    <span className="font-bold text-blue-400">{stock.symbol}</span>
                                    <span className="text-white">${stock.price}</span>
                                    <span className={stock.change >= 0 ? 'text-green-400' : 'text-red-400'}>
                                        {stock.change >= 0 ? '+' : ''}{stock.changePercent}%
                                    </span>
                                </div>
                            ))}
                        </div>
                    </div>

                    <div className="p-6">
                        {/* Header */}
                        <header className="flex justify-between items-center mb-8">
                            <div className="flex items-center">
                                <div className="bg-gradient-to-r from-blue-600 to-purple-600 p-3 rounded-xl mr-4">
                                    <icons.Activity size={24} />
                                </div>
                                <div>
                                    <h1 className="text-3xl font-bold gradient-text">TradeAI Pro</h1>
                                    <p className="text-slate-400">Advanced Trading Platform</p>
                                </div>
                            </div>
                            <div className="flex items-center space-x-4">
                                <div className="bg-slate-800 px-4 py-2 rounded-lg">
                                    <div className="text-sm text-slate-400">Portfolio</div>
                                    <div className="text-xl font-bold">${portfolioValue.toLocaleString()}</div>
                                </div>
                                <div className="bg-slate-800 px-4 py-2 rounded-lg">
                                    <div className="text-sm text-slate-400">Buying Power</div>
                                    <div className="text-xl font-bold text-green-400">${buyingPower.toLocaleString()}</div>
                                </div>
                                <button 
                                    onClick={() => setShowAIChat(true)}
                                    className="bg-blue-600 hover:bg-blue-700 px-4 py-2 rounded-lg flex items-center space-x-2"
                                >
                                    <icons.Brain size={18} />
                                    <span>AI Assistant</span>
                                </button>
                            </div>
                        </header>

                        {/* Navigation */}
                        <nav className="mb-8">
                            <div className="flex space-x-1 bg-slate-800 rounded-lg p-1">
                                {[
                                    { id: 'dashboard', label: 'Dashboard', icon: icons.BarChart },
                                    { id: 'trading', label: 'Trading', icon: icons.TrendingUp },
                                    { id: 'portfolio', label: 'Portfolio', icon: icons.DollarSign },
                                    { id: 'watchlist', label: 'Watchlist', icon: icons.Target },
                                    { id: 'orders', label: 'Orders', icon: icons.CreditCard }
                                ].map((tab) => (
                                    <button
                                        key={tab.id}
                                        onClick={() => setActiveTab(tab.id)}
                                        className={`flex items-center space-x-2 px-4 py-2 rounded-md transition-colors ${
                                            activeTab === tab.id 
                                                ? 'bg-blue-600 text-white' 
                                                : 'text-slate-400 hover:text-white hover:bg-slate-700'
                                        }`}
                                    >
                                        <tab.icon size={16} />
                                        <span>{tab.label}</span>
                                    </button>
                                ))}
                            </div>
                        </nav>

                        {/* Dashboard Tab */}
                        {activeTab === 'dashboard' && (
                            <div className="space-y-6">
                                {/* Portfolio Overview */}
                                <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
                                    <div className="bg-slate-800 p-6 rounded-xl glow-card">
                                        <div className="flex items-center justify-between">
                                            <div>
                                                <p className="text-slate-400 text-sm">Total Value</p>
                                                <p className="text-2xl font-bold">${portfolioValue.toLocaleString()}</p>
                                            </div>
                                            <icons.DollarSign className="text-blue-400" size={24} />
                                        </div>
                                        <p className={`text-sm mt-2 ${totalGainLoss >= 0 ? 'text-green-400' : 'text-red-400'}`}>
                                            {totalGainLoss >= 0 ? '+' : ''}${totalGainLoss.toFixed(2)} ({totalGainLossPercent.toFixed(2)}%)
                                        </p>
                                    </div>
                                    
                                    <div className="bg-slate-800 p-6 rounded-xl glow-card">
                                        <div className="flex items-center justify-between">
                                            <div>
                                                <p className="text-slate-400 text-sm">Day's Change</p>
                                                <p className="text-2xl font-bold text-green-400">+$1,234.56</p>
                                            </div>
                                            <icons.TrendingUp className="text-green-400" size={24} />
                                        </div>
                                        <p className="text-sm text-green-400 mt-2">+0.78% today</p>
                                    </div>
                                    
                                    <div className="bg-slate-800 p-6 rounded-xl glow-card">
                                        <div className="flex items-center justify-between">
                                            <div>
                                                <p className="text-slate-400 text-sm">Buying Power</p>
                                                <p className="text-2xl font-bold">${buyingPower.toLocaleString()}</p>
                                            </div>
                                            <icons.Target className="text-blue-400" size={24} />
                                        </div>
                                        <p className="text-sm text-slate-400 mt-2">Available to trade</p>
                                    </div>
                                    
                                    <div className="bg-slate-800 p-6 rounded-xl glow-card">
                                        <div className="flex items-center justify-between">
                                            <div>
                                                <p className="text-slate-400 text-sm">Total Positions</p>
                                                <p className="text-2xl font-bold">{positions.length}</p>
                                            </div>
                                            <icons.BarChart className="text-purple-400" size={24} />
                                        </div>
                                        <p className="text-sm text-slate-400 mt-2">Active holdings</p>
                                    </div>
                                </div>

                                {/* Market Overview & News */}
                                <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
                                    <div className="bg-slate-800 p-6 rounded-xl glow-card">
                                        <h3 className="text-xl font-bold mb-4">Top Movers</h3>
                                        <div className="space-y-3">
                                            {Object.values(stockData).sort((a, b) => Math.abs(b.changePercent) - Math.abs(a.changePercent)).slice(0, 5).map((stock, idx) => (
                                                <div key={idx} className="flex items-center justify-between">
                                                    <div>
                                                        <span className="font-medium">{stock.symbol}</span>
                                                        <span className="text-slate-400 text-sm ml-2">${stock.price}</span>
                                                    </div>
                                                    <div className={`flex items-center space-x-1 ${stock.change >= 0 ? 'text-green-400' : 'text-red-400'}`}>
                                                        {stock.change >= 0 ? <icons.TrendingUp size={16} /> : <icons.TrendingDown size={16} />}
                                                        <span>{stock.changePercent}%</span>
                                                    </div>
                                                </div>
                                            ))}
                                        </div>
                                    </div>

                                    <div className="bg-slate-800 p-6 rounded-xl glow-card">
                                        <h3 className="text-xl font-bold mb-4">Market News</h3>
                                        <div className="space-y-3">
                                            {news.map((item, idx) => (
                                                <div key={idx} className="border-l-2 border-blue-500 pl-3">
                                                    <p className="font-medium text-sm">{item.title}</p>
                                                    <div className="flex items-center space-x-2 text-xs text-slate-400 mt-1">
                                                        <span>{item.time}</span>
                                                        <span className={`px-2 py-1 rounded ${
                                                            item.impact === 'positive' ? 'bg-green-900 text-green-400' : 'bg-red-900 text-red-400'
                                                        }`}>
                                                            {item.impact}
                                                        </span>
                                                    </div>
                                                </div>
                                            ))}
                                        </div>
                                    </div>
                                </div>
                            </div>
                        )}

                        {/* Trading Tab */}
                        {activeTab === 'trading' && (
                            <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
                                <div className="lg:col-span-2 space-y-6">
                                    {/* Stock Search & Chart */}
                                    <div className="bg-slate-800 p-6 rounded-xl glow-card">
                                        <div className="flex justify-between items-center mb-4">
                                            <h3 className="text-xl font-bold">Market Data</h3>
                                            <div className="flex items-center space-x-2">
                                                <input
                                                    type="text"
                                                    placeholder="Search stocks..."
                                                    className="bg-slate-700 text-white px-3 py-2 rounded-lg text-sm"
                                                />
                                                <icons.Search className="text-slate-400" />
                                            </div>
                                        </div>
                                        
                                        {selectedStock ? (
                                            <div>
                                                <div className="flex items-center justify-between mb-4">
                                                    <div>
                                                        <h4 className="text-2xl font-bold">{stockData[selectedStock]?.name}</h4>
                                                        <p className="text-slate-400">{selectedStock}</p>
                                                    </div>
                                                    <div className="text-right">
                                                        <p className="text-2xl font-bold">${stockData[selectedStock]?.price}</p>
                                                        <p className={`text-sm ${stockData[selectedStock]?.change >= 0 ? 'text-green-400' : 'text-red-400'}`}>
                                                            {stockData[selectedStock]?.change >= 0 ? '+' : ''}{stockData[selectedStock]?.change} ({stockData[selectedStock]?.changePercent}%)
                                                        </p>
                                                    </div>
                                                </div>
                                                <div className="h-64 mb-4">
                                                    <canvas ref={chartRef} className="w-full h-full"></canvas>
                                                </div>
                                                <div className="flex space-x-2">
                                                    <button 
                                                        onClick={() => setShowTradeModal(true)}
                                                        className="bg-green-600 hover:bg-green-700 px-4 py-2 rounded-lg flex-1"
                                                    >
                                                        Buy
                                                    </button>
                                                    <button 
                                                        onClick={() => { setTradeType('sell'); setShowTradeModal(true); }}
                                                        className="bg-red-600 hover:bg-red-700 px-4 py-2 rounded-lg flex-1"
                                                    >
                                                        Sell
                                                    </button>
                                                </div>
                                            </div>
                                        ) : (
                                            <div className="text-center py-12 text-slate-400">
                                                Select a stock to view chart and trading options
                                            </div>
                                        )}
                                    </div>

                                    {/* Stock List */}
                                    <div className="bg-slate-800 p-6 rounded-xl glow-card">
                                        <h3 className="text-xl font-bold mb-4">Available Stocks</h3>
                                        <div className="space-y-2">
                                            {Object.values(stockData).map((stock, idx) => (
                                                <div 
                                                    key={idx} 
                                                    onClick={() => setSelectedStock(stock.symbol)}
                                                    className={`flex items-center justify-between p-3 rounded-lg cursor-pointer transition-colors ${
                                                        selectedStock === stock.symbol ? 'bg-blue-600' : 'hover:bg-slate-700'
                                                    }`}
                                                >
                                                    <div>
                                                        <span className="font-bold">{stock.symbol}</span>
                                                        <span className="text-slate-400 text-sm ml-2">{stock.name}</span>
                                                    </div>
                                                    <div className="text-right">
                                                        <div className="font-bold">${stock.price}</div>
                                                        <div className={`text-sm ${stock.change >= 0 ? 'text-green-400' : 'text-red-400'}`}>
                                                            {stock.change >= 0 ? '+' : ''}{stock.changePercent}%
                                                        </div>
                                                    </div>
                                                </div>
                                            ))}
                                        </div>
                                    </div>
                                </div>

                                {/* Trading Panel */}
                                <div className="space-y-6">
                                    <div className="bg-slate-800 p-6 rounded-xl glow-card">
                                        <h3 className="text-xl font-bold mb-4">Quick Trade</h3>
                                        {selectedStock ? (
                                            <div className="space-y-4">
                                                <div>
                                                    <label className="block text-sm font-medium mb-2">Stock</label>
                                                    <input 
                                                        type="text" 
                                                        value={selectedStock} 
                                                        readOnly 
                                                        className="w-full bg-slate-700 px-3 py-2 rounded-lg"
                                                    />
                                                </div>
                                                <div>
                                                    <label className="block text-sm font-medium mb-2">Quantity</label>
                                                    <input 
                                                        type="number" 
                                                        value={tradeQuantity}
                                                        onChange={(e) => setTradeQuantity(parseInt(e.target.value) || 1)}
                                                        className="w-full bg-slate-700 px-3 py-2 rounded-lg"
                                                        min="1"
                                                    />
                                                </div>
                                                <div>
                                                    <label className="block text-sm font-medium mb-2">Est. Cost</label>
                                                    <div className="text-xl font-bold text-blue-400">
                                                        ${(stockData[selectedStock]?.price * tradeQuantity || 0).toFixed(2)}
                                                    </div>
                                                </div>
                                                <div className="flex space-x-2">
                                                    <button 
                                                        onClick={() => { setTradeType('buy'); executeTrade(); }}
                                                        className="bg-green-600 hover:bg-green-700 px-4 py-2 rounded-lg flex-1"
                                                    >
                                                        Buy
                                                    </button>
                                                    <button 
                                                        onClick={() => { setTradeType('sell'); executeTrade(); }}
                                                        className="bg-red-600 hover:bg-red-700 px-4 py-2 rounded-lg flex-1"
                                                    >
                                                        Sell
                                                    </button>
                                                </div>
                                            </div>
                                        ) : (
                                            <p className="text-slate-400 text-center py-8">Select a stock to trade</p>
                                        )}
                                    </div>

                                    {/* Account Summary */}
                                    <div className="bg-slate-800 p-6 rounded-xl glow-card">
                                        <h3 className="text-xl font-bold mb-4">Account Summary</h3>
                                        <div className="space-y-3">
                                            <div className="flex justify-between">
                                                <span className="text-slate-400">Portfolio Value</span>
                                                <span className="font-bold">${portfolioValue.toLocaleString()}</span>
                                            </div>
                                            <div className="flex justify-between">
                                                <span className="text-slate-400">Buying Power</span>
                                                <span className="font-bold text-green-400">${buyingPower.toLocaleString()}</span>
                                            </div>
                                            <div className="flex justify-between">
                                                <span className="text-slate-400">Day's P&L</span>
                                                <span className="font-bold text-green-400">+$1,234.56</span>
                                            </div>
                                            <div className="flex justify-between">
                                                <span className="text-slate-400">Total P&L</span>
                                                <span className={`font-bold ${totalGainLoss >= 0 ? 'text-green-400' : 'text-red-400'}`}>
                                                    {totalGainLoss >= 0 ? '+' : ''}${totalGainLoss.toFixed(2)}
                                                </span>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        )}

                        {/* Portfolio Tab */}
                        {activeTab === 'portfolio' && (
                            <div className="space-y-6">
                                <div className="bg-slate-800 p-6 rounded-xl glow-card">
                                    <h3 className="text-xl font-bold mb-4">Your Positions</h3>
                                    <div className="overflow-x-auto">
                                        <table className="w-full">
                                            <thead>
                                                <tr className="text-left border-b border-slate-700">
                                                    <th className="py-3">Symbol</th>
                                                    <th className="py-3">Shares</th>
                                                    <th className="py-3">Avg Price</th>
                                                    <th className="py-3">Current Price</th>
                                                    <th className="py-3">Value</th>
                                                    <th className="py-3">P&L</th>
                                                    <th className="py-3">Actions</th>
                                                </tr>
                                            </thead>
                                            <tbody>
                                                {positions.map((position, idx) => {
                                                    const pnl = (position.currentPrice - position.avgPrice) * position.shares;
                                                    const pnlPercent = ((position.currentPrice - position.avgPrice) / position.avgPrice) * 100;
                                                    return (
                                                        <tr key={idx} className="border-b border-slate-700">
                                                            <td className="py-3 font-bold text-blue-400">{position.symbol}</td>
                                                            <td className="py-3">{position.shares}</td>
                                                            <td className="py-3">${position.avgPrice.toFixed(2)}</td>
                                                            <td className="py-3">${position.currentPrice.toFixed(2)}</td>
                                                            <td className="py-3 font-bold">${position.value.toFixed(2)}</td>
                                                            <td className={`py-3 font-bold ${pnl >= 0 ? 'text-green-400' : 'text-red-400'}`}>
                                                                {pnl >= 0 ? '+' : ''}${pnl.toFixed(2)}
                                                                <div className="text-xs">({pnlPercent.toFixed(2)}%)</div>
                                                            </td>
                                                            <td className="py-3">
                                                                <button 
                                                                    onClick={() => { setSelectedStock(position.symbol); setShowTradeModal(true); }}
                                                                    className="bg-blue-600 hover:bg-blue-700 px-3 py-1 rounded text-sm"
                                                                >
                                                                    Trade
                                                                </button>
                                                            </td>
                                                        </tr>
                                                    );
                                                })}
                                            </tbody>
                                        </table>
                                    </div>
                                </div>

                                {/* Portfolio Allocation */}
                                <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
                                    <div className="bg-slate-800 p-6 rounded-xl glow-card">
                                        <h3 className="text-xl font-bold mb-4">Portfolio Allocation</h3>
                                        <div className="space-y-3">
                                            {positions.map((position, idx) => {
                                                const percentage = (position.value / portfolioValue) * 100;
                                                return (
                                                    <div key={idx}>
                                                        <div className="flex justify-between mb-1">
                                                            <span className="font-medium">{position.symbol}</span>
                                                            <span>{percentage.toFixed(1)}%</span>
                                                        </div>
                                                        <div className="w-full bg-slate-700 rounded-full h-2">
                                                            <div 
                                                                className="bg-blue-600 h-2 rounded-full transition-all duration-300"
                                                                style={{ width: `${percentage}%` }}
                                                            ></div>
                                                        </div>
                                                    </div>
                                                );
                                            })}
                                        </div>
                                    </div>

                                    <div className="bg-slate-800 p-6 rounded-xl glow-card">
                                        <h3 className="text-xl font-bold mb-4">Performance Metrics</h3>
                                        <div className="space-y-4">
                                            <div className="flex justify-between">
                                                <span className="text-slate-400">Total Return</span>
                                                <span className={`font-bold ${totalGainLoss >= 0 ? 'text-green-400' : 'text-red-400'}`}>
                                                    {totalGainLossPercent.toFixed(2)}%
                                                </span>
                                            </div>
                                            <div className="flex justify-between">
                                                <span className="text-slate-400">Best Performer</span>
                                                <span className="font-bold text-green-400">NVDA (+10.6%)</span>
                                            </div>
                                            <div className="flex justify-between">
                                                <span className="text-slate-400">Worst Performer</span>
                                                <span className="font-bold text-red-400">TSLA (-2.8%)</span>
                                            </div>
                                            <div className="flex justify-between">
                                                <span className="text-slate-400">Risk Score</span>
                                                <span className="font-bold text-yellow-400">Medium</span>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        )}

                        {/* Watchlist Tab */}
                        {activeTab === 'watchlist' && (
                            <div className="space-y-6">
                                <div className="bg-slate-800 p-6 rounded-xl glow-card">
                                    <div className="flex justify-between items-center mb-4">
                                        <h3 className="text-xl font-bold">Watchlist</h3>
                                        <button className="bg-blue-600 hover:bg-blue-700 px-4 py-2 rounded-lg flex items-center space-x-2">
                                            <icons.Plus size={16} />
                                            <span>Add Stock</span>
                                        </button>
                                    </div>
                                    <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
                                        {watchlist.map((symbol, idx) => {
                                            const stock = stockData[symbol];
                                            if (!stock) return null;
                                            return (
                                                <div key={idx} className="bg-slate-700 p-4 rounded-lg glow-card">
                                                    <div className="flex justify-between items-start mb-2">
                                                        <div>
                                                            <h4 className="font-bold text-blue-400">{stock.symbol}</h4>
                                                            <p className="text-xs text-slate-400">{stock.name}</p>
                                                        </div>
                                                        <button 
                                                            onClick={() => setWatchlist(prev => prev.filter(s => s !== symbol))}
                                                            className="text-slate-400 hover:text-red-400"
                                                        >
                                                            ×
                                                        </button>
                                                    </div>
                                                    <div className="mb-3">
                                                        <p className="text-xl font-bold">${stock.price}</p>
                                                        <p className={`text-sm ${stock.change >= 0 ? 'text-green-400' : 'text-red-400'}`}>
                                                            {stock.change >= 0 ? '+' : ''}{stock.change} ({stock.changePercent}%)
                                                        </p>
                                                    </div>
                                                    <div className="flex space-x-2">
                                                        <button 
                                                            onClick={() => { setSelectedStock(symbol); setShowTradeModal(true); }}
                                                            className="bg-green-600 hover:bg-green-700 px-3 py-1 rounded text-sm flex-1"
                                                        >
                                                            Buy
                                                        </button>
                                                        <button 
                                                            onClick={() => { setSelectedStock(symbol); setActiveTab('trading'); }}
                                                            className="bg-blue-600 hover:bg-blue-700 px-3 py-1 rounded text-sm flex-1"
                                                        >
                                                            Chart
                                                        </button>
                                                    </div>
                                                </div>
                                            );
                                        })}
                                    </div>
                                </div>
                            </div>
                        )}

                        {/* Orders Tab */}
                        {activeTab === 'orders' && (
                            <div className="bg-slate-800 p-6 rounded-xl glow-card">
                                <h3 className="text-xl font-bold mb-4">Order History</h3>
                                {orders.length > 0 ? (
                                    <div className="overflow-x-auto">
                                        <table className="w-full">
                                            <thead>
                                                <tr className="text-left border-b border-slate-700">
                                                    <th className="py-3">Symbol</th>
                                                    <th className="py-3">Type</th>
                                                    <th className="py-3">Quantity</th>
                                                    <th className="py-3">Price</th>
                                                    <th className="py-3">Total</th>
                                                    <th className="py-3">Status</th>
                                                    <th className="py-3">Time</th>
                                                </tr>
                                            </thead>
                                            <tbody>
                                                {orders.map((order, idx) => (
                                                    <tr key={idx} className="border-b border-slate-700">
                                                        <td className="py-3 font-bold text-blue-400">{order.symbol}</td>
                                                        <td className={`py-3 font-medium ${
                                                            order.type === 'buy' ? 'text-green-400' : 'text-red-400'
                                                        }`}>
                                                            {order.type.toUpperCase()}
                                                        </td>
                                                        <td className="py-3">{order.quantity}</td>
                                                        <td className="py-3">${order.price.toFixed(2)}</td>
                                                        <td className="py-3 font-bold">${(order.quantity * order.price).toFixed(2)}</td>
                                                        <td className="py-3">
                                                            <span className="bg-green-900 text-green-400 px-2 py-1 rounded-full text-xs">
                                                                {order.status}
                                                            </span>
                                                        </td>
                                                        <td className="py-3 text-sm text-slate-400">
                                                            {order.timestamp.toLocaleString()}
                                                        </td>
                                                    </tr>
                                                ))}
                                            </tbody>
                                        </table>
                                    </div>
                                ) : (
                                    <div className="text-center py-12 text-slate-400">
                                        <icons.CreditCard size={48} className="mx-auto mb-4 opacity-50" />
                                        <p>No orders yet. Start trading to see your order history.</p>
                                    </div>
                                )}
                            </div>
                        )}

                        {/* Trading Modal */}
                        {showTradeModal && (
                            <div className="fixed inset-0 bg-black bg-opacity-75 flex items-center justify-center z-50 p-4">
                                <div className="bg-slate-800 text-white rounded-xl shadow-2xl w-full max-w-md">
                                    <div className="p-6 border-b border-slate-700">
                                        <div className="flex justify-between items-center">
                                            <h3 className="text-xl font-bold">
                                                {tradeType === 'buy' ? 'Buy' : 'Sell'} {selectedStock}
                                            </h3>
                                            <button 
                                                onClick={() => setShowTradeModal(false)}
                                                className="text-slate-400 hover:text-white text-2xl"
                                            >
                                                ×
                                            </button>
                                        </div>
                                        {selectedStock && stockData[selectedStock] && (
                                            <div className="mt-2">
                                                <p className="text-2xl font-bold">${stockData[selectedStock].price}</p>
                                                <p className={`text-sm ${stockData[selectedStock].change >= 0 ? 'text-green-400' : 'text-red-400'}`}>
                                                    {stockData[selectedStock].change >= 0 ? '+' : ''}{stockData[selectedStock].change} ({stockData[selectedStock].changePercent}%)
                                                </p>
                                            </div>
                                        )}
                                    </div>
                                    
                                    <div className="p-6 space-y-4">
                                        <div className="flex space-x-2">
                                            <button
                                                onClick={() => setTradeType('buy')}
                                                className={`flex-1 py-2 px-4 rounded-lg font-medium transition-colors ${
                                                    tradeType === 'buy' 
                                                        ? 'bg-green-600 text-white' 
                                                        : 'bg-slate-700 text-slate-300 hover:bg-slate-600'
                                                }`}
                                            >
                                                Buy
                                            </button>
                                            <button
                                                onClick={() => setTradeType('sell')}
                                                className={`flex-1 py-2 px-4 rounded-lg font-medium transition-colors ${
                                                    tradeType === 'sell' 
                                                        ? 'bg-red-600 text-white' 
                                                        : 'bg-slate-700 text-slate-300 hover:bg-slate-600'
                                                }`}
                                            >
                                                Sell
                                            </button>
                                        </div>

                                        <div>
                                            <label className="block text-sm font-medium mb-2">Quantity</label>
                                            <input
                                                type="number"
                                                value={tradeQuantity}
                                                onChange={(e) => setTradeQuantity(parseInt(e.target.value) || 1)}
                                                className="w-full bg-slate-700 text-white px-4 py-3 rounded-lg border border-slate-600 focus:border-blue-500 focus:outline-none"
                                                min="1"
                                            />
                                        </div>

                                        <div className="bg-slate-700 p-4 rounded-lg">
                                            <div className="flex justify-between items-center">
                                                <span>Estimated Total:</span>
                                                <span className="text-xl font-bold">
                                                    ${selectedStock && stockData[selectedStock] 
                                                        ? (stockData[selectedStock].price * tradeQuantity).toFixed(2) 
                                                        : '0.00'}
                                                </span>
                                            </div>
                                        </div>

                                        <button 
                                            onClick={executeTrade}
                                            className={`w-full py-3 px-4 rounded-lg font-bold text-white transition-colors ${
                                                tradeType === 'buy' 
                                                    ? 'bg-green-600 hover:bg-green-700' 
                                                    : 'bg-red-600 hover:bg-red-700'
                                            }`}
                                        >
                                            {tradeType === 'buy' ? 'Place Buy Order' : 'Place Sell Order'}
                                        </button>
                                    </div>
                                </div>
                            </div>
                        )}

                        {/* AI Chat Modal */}
                        {showAIChat && (
                            <div className="fixed inset-0 bg-black bg-opacity-75 flex items-center justify-center z-50 p-4">
                                <div className="bg-slate-800 rounded-xl shadow-2xl w-full max-w-md max-h-[80vh] flex flex-col">
                                    <div className="p-4 border-b border-slate-700 flex justify-between items-center">
                                        <div className="flex items-center space-x-2">
                                            <icons.Brain className="text-blue-400" />
                                            <h3 className="font-bold text-white">AI Trading Assistant</h3>
                                        </div>
                                        <button onClick={() => setShowAIChat(false)} className="text-slate-400 hover:text-white text-2xl">
                                            ×
                                        </button>
                                    </div>
                                    
                                    <div className="flex-1 overflow-y-auto p-4 space-y-4">
                                        {conversation.length === 0 ? (
                                            <div className="text-center text-slate-400 py-8">
                                                <icons.Brain size={48} className="mx-auto mb-4 text-blue-400" />
                                                <p>Hello! I'm your AI trading assistant. Ask me about market trends, trading strategies, or your portfolio.</p>
                                            </div>
                                        ) : (
                                            conversation.map((msg, index) => (
                                                <div key={index} className={`flex ${msg.role === 'user' ? 'justify-end' : 'justify-start'}`}>
                                                    <div className={`max-w-xs p-3 rounded-lg ${
                                                        msg.role === 'user' 
                                                            ? 'bg-blue-600 text-white' 
                                                            : 'bg-slate-700 text-white'
                                                    }`}>
                                                        {msg.content}
                                                    </div>
                                                </div>
                                            ))
                                        )}
                                    </div>
                                    
                                    <form onSubmit={handleChatSubmit} className="p-4 border-t border-slate-700">
                                        <div className="flex space-x-2">
                                            <input
                                                type="text"
                                                value={chatMessage}
                                                onChange={(e) => setChatMessage(e.target.value)}
                                                placeholder="Ask about trading strategies..."
                                                className="flex-1 bg-slate-700 text-white px-4 py-2 rounded-lg border border-slate-600 focus:border-blue-500 focus:outline-none"
                                            />
                                            <button 
                                                type="submit"
                                                className="bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded-lg transition-colors"
                                            >
                                                Send
                                            </button>
                                        </div>
                                    </form>
                                </div>
                            </div>
                        )}
                    </div>
                </div>
            );
        };

        ReactDOM.render(<TradeAIPro />, document.getElementById('root'));
    </script>
</body>
</html>
