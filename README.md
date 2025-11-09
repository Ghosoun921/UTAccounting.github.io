import React, { useState, useEffect } from 'react';
import { CheckCircle, XCircle, RotateCcw, Trophy, Star, Zap, Target, TrendingUp } from 'lucide-react';

const DebitCreditGame = () => {
  const allItems = [
    { id: 1, name: 'Cash Received from Customer', value: 500, correctSide: 'debit', category: 'Asset', hint: 'Increases cash (asset)' },
    { id: 2, name: 'Bank Loan Obtained', value: 1000, correctSide: 'credit', category: 'Liability', hint: 'Money owed to bank' },
    { id: 3, name: 'Equipment Purchase', value: 800, correctSide: 'debit', category: 'Asset', hint: 'Acquiring an asset' },
    { id: 4, name: 'Salary Expense', value: 300, correctSide: 'debit', category: 'Expense', hint: 'Cost of doing business' },
    { id: 5, name: 'Service Revenue', value: 1200, correctSide: 'credit', category: 'Revenue', hint: 'Income earned' },
    { id: 6, name: 'Office Supplies Purchased', value: 150, correctSide: 'debit', category: 'Asset', hint: 'Supplies on hand' },
    { id: 7, name: 'Accounts Payable', value: 400, correctSide: 'credit', category: 'Liability', hint: 'Money owed to suppliers' },
    { id: 8, name: 'Owner Capital Investment', value: 2000, correctSide: 'credit', category: 'Equity', hint: 'Owner puts money in' },
    { id: 9, name: 'Inventory Purchased', value: 600, correctSide: 'debit', category: 'Asset', hint: 'Goods for resale' },
    { id: 10, name: 'Rent Expense', value: 500, correctSide: 'debit', category: 'Expense', hint: 'Cost of office space' },
    { id: 11, name: 'Sales Revenue', value: 750, correctSide: 'credit', category: 'Revenue', hint: 'Money from sales' },
    { id: 12, name: 'Utilities Expense', value: 200, correctSide: 'debit', category: 'Expense', hint: 'Power and water costs' },
    { id: 13, name: 'Notes Payable', value: 1500, correctSide: 'credit', category: 'Liability', hint: 'Formal debt obligation' },
    { id: 14, name: 'Advertising Expense', value: 350, correctSide: 'debit', category: 'Expense', hint: 'Marketing costs' },
    { id: 15, name: 'Accounts Receivable', value: 450, correctSide: 'debit', category: 'Asset', hint: 'Money customers owe you' },
  ];

  const [items, setItems] = useState([]);
  const [debitItems, setDebitItems] = useState([]);
  const [creditItems, setCreditItems] = useState([]);
  const [score, setScore] = useState(0);
  const [streak, setStreak] = useState(0);
  const [bestStreak, setBestStreak] = useState(0);
  const [gameOver, setGameOver] = useState(false);
  const [feedback, setFeedback] = useState('');
  const [difficulty, setDifficulty] = useState('medium');
  const [showHints, setShowHints] = useState(false);
  const [correctCount, setCorrectCount] = useState(0);
  const [totalMoves, setTotalMoves] = useState(0);
  const [timer, setTimer] = useState(0);
  const [timerActive, setTimerActive] = useState(false);

  useEffect(() => {
    startNewGame();
  }, [difficulty]);

  useEffect(() => {
    let interval;
    if (timerActive && !gameOver) {
      interval = setInterval(() => {
        setTimer(t => t + 1);
      }, 1000);
    }
    return () => clearInterval(interval);
  }, [timerActive, gameOver]);

  const getDifficultyItemCount = () => {
    switch(difficulty) {
      case 'easy': return 4;
      case 'medium': return 6;
      case 'hard': return 8;
      default: return 6;
    }
  };

  const startNewGame = () => {
    const itemCount = getDifficultyItemCount();
    const shuffled = [...allItems].sort(() => Math.random() - 0.5).slice(0, itemCount);
    setItems(shuffled);
    setDebitItems([]);
    setCreditItems([]);
    setScore(0);
    setStreak(0);
    setGameOver(false);
    setFeedback('');
    setCorrectCount(0);
    setTotalMoves(0);
    setTimer(0);
    setTimerActive(true);
  };

  const placeItem = (item, side) => {
    const newItems = items.filter(i => i.id !== item.id);
    setItems(newItems);
    setTotalMoves(totalMoves + 1);

    if (side === 'debit') {
      setDebitItems([...debitItems, item]);
    } else {
      setCreditItems([...creditItems, item]);
    }

    const isCorrect = item.correctSide === side;
    
    if (isCorrect) {
      const newStreak = streak + 1;
      setStreak(newStreak);
      setBestStreak(Math.max(bestStreak, newStreak));
      const points = 10 + (newStreak >= 3 ? 5 : 0);
      setScore(score + points);
      setCorrectCount(correctCount + 1);
      setFeedback(newStreak >= 3 ? 
        `🔥 ${points} points! ${newStreak}x streak!` : 
        `✓ Correct! +${points} points`
      );
    } else {
      setStreak(0);
      setScore(Math.max(0, score - 5));
      setFeedback(`✗ Wrong! ${item.name} is a ${item.category} (${item.correctSide} side)`);
    }

    setTimeout(() => setFeedback(''), 2500);

    if (newItems.length === 0) {
      setGameOver(true);
      setTimerActive(false);
    }
  };

  const getTotal = (itemList) => {
    return itemList.reduce((sum, item) => sum + item.value, 0);
  };

  const debitTotal = getTotal(debitItems);
  const creditTotal = getTotal(creditItems);
  const isBalanced = gameOver && debitTotal === creditTotal;
  const accuracy = totalMoves > 0 ? Math.round((correctCount / totalMoves) * 100) : 0;

  const formatTime = (seconds) => {
    const mins = Math.floor(seconds / 60);
    const secs = seconds % 60;
    return `${mins}:${secs.toString().padStart(2, '0')}`;
  };

  const getCategoryColor = (category) => {
    switch(category) {
      case 'Asset': return 'text-blue-600';
      case 'Liability': return 'text-red-600';
      case 'Equity': return 'text-purple-600';
      case 'Revenue': return 'text-green-600';
      case 'Expense': return 'text-orange-600';
      default: return 'text-gray-600';
    }
  };

  return (
    <div className="min-h-screen bg-gradient-to-br from-slate-900 via-purple-900 to-slate-900 p-4 md:p-8">
      <div className="max-w-7xl mx-auto">
        {/* Header */}
        <div className="text-center mb-6">
          <h1 className="text-4xl md:text-5xl font-bold text-white mb-3 flex items-center justify-center gap-3">
            <Zap className="w-10 h-10 text-yellow-400" />
            Debit & Credit Challenge
            <Zap className="w-10 h-10 text-yellow-400" />
          </h1>
          <p className="text-gray-300 text-lg">Master the fundamentals of accounting!</p>
        </div>

        {/* Stats Bar */}
        <div className="grid grid-cols-2 md:grid-cols-5 gap-3 mb-6">
          <div className="bg-gradient-to-br from-blue-500 to-blue-600 rounded-lg p-4 text-white shadow-lg">
            <div className="text-sm opacity-90">Score</div>
            <div className="text-2xl font-bold flex items-center gap-2">
              <Star className="w-5 h-5" />
              {score}
            </div>
          </div>
          <div className="bg-gradient-to-br from-orange-500 to-orange-600 rounded-lg p-4 text-white shadow-lg">
            <div className="text-sm opacity-90">Streak</div>
            <div className="text-2xl font-bold flex items-center gap-2">
              <TrendingUp className="w-5 h-5" />
              {streak}x
            </div>
          </div>
          <div className="bg-gradient-to-br from-purple-500 to-purple-600 rounded-lg p-4 text-white shadow-lg">
            <div className="text-sm opacity-90">Best Streak</div>
            <div className="text-2xl font-bold">{bestStreak}x</div>
          </div>
          <div className="bg-gradient-to-br from-green-500 to-green-600 rounded-lg p-4 text-white shadow-lg">
            <div className="text-sm opacity-90">Accuracy</div>
            <div className="text-2xl font-bold flex items-center gap-2">
              <Target className="w-5 h-5" />
              {accuracy}%
            </div>
          </div>
          <div className="bg-gradient-to-br from-pink-500 to-pink-600 rounded-lg p-4 text-white shadow-lg">
            <div className="text-sm opacity-90">Time</div>
            <div className="text-2xl font-bold">{formatTime(timer)}</div>
          </div>
        </div>

        {/* Controls */}
        <div className="flex flex-wrap justify-center gap-3 mb-6">
          <div className="bg-white/10 backdrop-blur-sm rounded-lg p-2 flex gap-2">
            <button
              onClick={() => setDifficulty('easy')}
              className={`px-4 py-2 rounded-lg font-semibold transition ${difficulty === 'easy' ? 'bg-green-500 text-white' : 'bg-white/20 text-white hover:bg-white/30'}`}
            >
              Easy (4)
            </button>
            <button
              onClick={() => setDifficulty('medium')}
              className={`px-4 py-2 rounded-lg font-semibold transition ${difficulty === 'medium' ? 'bg-yellow-500 text-white' : 'bg-white/20 text-white hover:bg-white/30'}`}
            >
              Medium (6)
            </button>
            <button
              onClick={() => setDifficulty('hard')}
              className={`px-4 py-2 rounded-lg font-semibold transition ${difficulty === 'hard' ? 'bg-red-500 text-white' : 'bg-white/20 text-white hover:bg-white/30'}`}
            >
              Hard (8)
            </button>
          </div>
          <button
            onClick={() => setShowHints(!showHints)}
            className={`px-6 py-2 rounded-lg font-semibold transition ${showHints ? 'bg-purple-500 text-white' : 'bg-white/20 text-white hover:bg-white/30'}`}
          >
            {showHints ? '🔍 Hints On' : '💡 Show Hints'}
          </button>
        </div>

        {/* Feedback */}
        {feedback && (
          <div className={`text-center mb-4 p-4 rounded-lg font-bold text-lg animate-pulse ${feedback.includes('✓') || feedback.includes('🔥') ? 'bg-green-500/20 text-green-300 border-2 border-green-500' : 'bg-red-500/20 text-red-300 border-2 border-red-500'}`}>
            {feedback}
          </div>
        )}

        <div className="grid grid-cols-1 lg:grid-cols-3 gap-6 mb-6">
          {/* Available Items */}
          <div className="bg-white/10 backdrop-blur-md rounded-xl shadow-2xl p-6 border border-white/20">
            <h2 className="text-2xl font-bold text-white mb-4 text-center flex items-center justify-center gap-2">
              <Zap className="w-6 h-6 text-yellow-400" />
              Transactions
            </h2>
            <div className="space-y-3 max-h-[600px] overflow-y-auto">
              {items.map(item => (
                <div key={item.id} className="bg-gradient-to-br from-yellow-400 to-orange-400 rounded-lg p-4 shadow-lg transform hover:scale-105 transition">
                  <div className="font-bold text-gray-900">{item.name}</div>
                  <div className="text-sm text-gray-800 mt-1 font-semibold">${item.value.toLocaleString()}</div>
                  <div className={`text-xs mt-1 font-semibold ${getCategoryColor(item.category)}`}>
                    {item.category}
                  </div>
                  {showHints && (
                    <div className="text-xs text-gray-700 mt-2 italic bg-white/50 p-2 rounded">
                      💡 {item.hint}
                    </div>
                  )}
                  <div className="flex gap-2 mt-3">
                    <button
                      onClick={() => placeItem(item, 'debit')}
                      className="flex-1 bg-blue-600 hover:bg-blue-700 text-white py-2 px-3 rounded-lg text-sm font-bold transition shadow-md"
                    >
                      ← DEBIT
                    </button>
                    <button
                      onClick={() => placeItem(item, 'credit')}
                      className="flex-1 bg-green-600 hover:bg-green-700 text-white py-2 px-3 rounded-lg text-sm font-bold transition shadow-md"
                    >
                      CREDIT →
                    </button>
                  </div>
                </div>
              ))}
              {items.length === 0 && !gameOver && (
                <div className="text-center text-white/60 py-8">
                  All items placed!
                </div>
              )}
            </div>
          </div>

          {/* Debit Side */}
          <div className="bg-gradient-to-br from-blue-900/50 to-blue-800/50 backdrop-blur-md rounded-xl shadow-2xl p-6 border-2 border-blue-400">
            <h2 className="text-2xl font-bold text-blue-300 mb-4 text-center">DEBIT SIDE</h2>
            <div className="space-y-3 min-h-[400px]">
              {debitItems.map(item => (
                <div key={item.id} className={`rounded-lg p-4 shadow-md transition transform hover:scale-105 ${item.correctSide === 'debit' ? 'bg-green-500/30 border-2 border-green-400' : 'bg-red-500/30 border-2 border-red-400'}`}>
                  <div className="flex items-start justify-between">
                    <div className="flex-1">
                      <div className="font-semibold text-white">{item.name}</div>
                      <div className="text-sm text-white/80">${item.value.toLocaleString()}</div>
                      <div className={`text-xs mt-1 font-semibold ${getCategoryColor(item.category)}`}>
                        {item.category}
                      </div>
                    </div>
                    {item.correctSide === 'debit' ? (
                      <CheckCircle className="text-green-400 w-6 h-6 flex-shrink-0" />
                    ) : (
                      <XCircle className="text-red-400 w-6 h-6 flex-shrink-0" />
                    )}
                  </div>
                </div>
              ))}
            </div>
            <div className="mt-4 pt-4 border-t-2 border-blue-400">
              <div className="text-xl font-bold text-blue-300">Total: ${debitTotal.toLocaleString()}</div>
            </div>
          </div>

          {/* Credit Side */}
          <div className="bg-gradient-to-br from-green-900/50 to-green-800/50 backdrop-blur-md rounded-xl shadow-2xl p-6 border-2 border-green-400">
            <h2 className="text-2xl font-bold text-green-300 mb-4 text-center">CREDIT SIDE</h2>
            <div className="space-y-3 min-h-[400px]">
              {creditItems.map(item => (
                <div key={item.id} className={`rounded-lg p-4 shadow-md transition transform hover:scale-105 ${item.correctSide === 'credit' ? 'bg-green-500/30 border-2 border-green-400' : 'bg-red-500/30 border-2 border-red-400'}`}>
                  <div className="flex items-start justify-between">
                    <div className="flex-1">
                      <div className="font-semibold text-white">{item.name}</div>
                      <div className="text-sm text-white/80">${item.value.toLocaleString()}</div>
                      <div className={`text-xs mt-1 font-semibold ${getCategoryColor(item.category)}`}>
                        {item.category}
                      </div>
                    </div>
                    {item.correctSide === 'credit' ? (
                      <CheckCircle className="text-green-400 w-6 h-6 flex-shrink-0" />
                    ) : (
                      <XCircle className="text-red-400 w-6 h-6 flex-shrink-0" />
                    )}
                  </div>
                </div>
              ))}
            </div>
            <div className="mt-4 pt-4 border-t-2 border-green-400">
              <div className="text-xl font-bold text-green-300">Total: ${creditTotal.toLocaleString()}</div>
            </div>
          </div>
        </div>

        {/* Game Over */}
        {gameOver && (
          <div className="text-center">
            <div className={`inline-block p-8 rounded-xl shadow-2xl ${isBalanced ? 'bg-gradient-to-br from-green-500 to-emerald-600' : 'bg-gradient-to-br from-blue-500 to-indigo-600'}`}>
              {isBalanced ? (
                <>
                  <Trophy className="w-16 h-16 text-yellow-300 mx-auto mb-4 animate-bounce" />
                  <h2 className="text-3xl font-bold text-white mb-3">🎉 PERFECT BALANCE! 🎉</h2>
                  <p className="text-white text-lg mb-2">Debit and Credit sides are equal!</p>
                  <p className="text-white/90">Both sides: ${debitTotal.toLocaleString()}</p>
                </>
              ) : (
                <>
                  <Star className="w-16 h-16 text-yellow-300 mx-auto mb-4" />
                  <h2 className="text-3xl font-bold text-white mb-3">Game Complete!</h2>
                  <p className="text-white text-lg">Debit: ${debitTotal.toLocaleString()} | Credit: ${creditTotal.toLocaleString()}</p>
                  <p className="text-white/80 mt-2">Difference: ${Math.abs(debitTotal - creditTotal).toLocaleString()}</p>
                </>
              )}
              <div className="mt-6 space-y-2 text-white">
                <div className="text-2xl font-bold">Final Score: {score}</div>
                <div>Accuracy: {accuracy}%</div>
                <div>Best Streak: {bestStreak}x</div>
                <div>Time: {formatTime(timer)}</div>
              </div>
            </div>
          </div>
        )}

        {/* New Game Button */}
        <div className="text-center mt-6">
          <button
            onClick={startNewGame}
            className="bg-gradient-to-r from-purple-600 to-pink-600 hover:from-purple-700 hover:to-pink-700 text-white py-4 px-10 rounded-xl font-bold text-lg transition transform hover:scale-105 shadow-lg flex items-center gap-3 mx-auto"
          >
            <RotateCcw className="w-6 h-6" />
            New Game
          </button>
        </div>

        {/* Reference Guide */}
        <div className="mt-8 bg-white/10 backdrop-blur-md rounded-xl shadow-2xl p-6 border border-white/20">
          <h3 className="text-2xl font-bold text-white mb-4 text-center">📚 Accounting Guide</h3>
          <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
            <div className="bg-blue-900/30 p-4 rounded-lg border border-blue-400">
              <h4 className="font-bold text-blue-300 mb-3 text-lg">DEBIT increases:</h4>
              <ul className="space-y-2 text-white">
                <li className="flex items-start gap-2">
                  <span className="text-blue-400 font-bold">•</span>
                  <span><strong className="text-blue-400">Assets:</strong> Cash, Equipment, Inventory, Accounts Receivable</span>
                </li>
                <li className="flex items-start gap-2">
                  <span className="text-orange-400 font-bold">•</span>
                  <span><strong className="text-orange-400">Expenses:</strong> Rent, Salaries, Utilities, Advertising</span>
                </li>
              </ul>
            </div>
            <div className="bg-green-900/30 p-4 rounded-lg border border-green-400">
              <h4 className="font-bold text-green-300 mb-3 text-lg">CREDIT increases:</h4>
              <ul className="space-y-2 text-white">
                <li className="flex items-start gap-2">
                  <span className="text-red-400 font-bold">•</span>
                  <span><strong className="text-red-400">Liabilities:</strong> Loans, Accounts Payable, Notes Payable</span>
                </li>
                <li className="flex items-start gap-2">
                  <span className="text-purple-400 font-bold">•</span>
                  <span><strong className="text-purple-400">Equity:</strong> Owner Investment, Retained Earnings</span>
                </li>
                <li className="flex items-start gap-2">
                  <span className="text-green-400 font-bold">•</span>
                  <span><strong className="text-green-400">Revenue:</strong> Sales, Service Income</span>
                </li>
              </ul>
            </div>
          </div>
          <div className="mt-4 text-center text-white/80 text-sm">
            💡 Tip: Get 3+ correct in a row for streak bonus points!
          </div>
        </div>
      </div>
    </div>
  );
};

export default DebitCreditGame;
