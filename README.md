'use client';

import React, { useState, useEffect } from 'react';

export default function Home() {
const [isStarted, setIsStarted] = useState(false);
const [currentStep, setCurrentStep] = useState(0);
const [message, setMessage] = useState('');
const [isEditing, setIsEditing] = useState(false);
const [newItemName, setNewItemName] = useState('');
const [weather, setWeather] = useState('sunny'); // sunny, cloudy, rainy

// LocalStorage から項目を読み込む
const [items, setItems] = useState(() => {
if (typeof window !== 'undefined') {
const saved = localStorage.getItem('checklist-items');
if (saved) {
return JSON.parse(saved);
}
}
return [
{ name: "財布", id: 1, emoji: "👛", required: true },
{ name: "鍵", id: 2, emoji: "🔑", required: true },
{ name: "スマートフォン", id: 3, emoji: "📱", required: true }
];
});

useEffect(() => {
if (typeof window !== 'undefined') {
localStorage.setItem('checklist-items', JSON.stringify(items));
}
}, [items]);

const addItem = () => {
if (newItemName.trim()) {
setItems([
...items,
{
name: newItemName,
id: Date.now(),
emoji: "📦",
required: false
}
]);
setNewItemName('');
}
};

const removeItem = (id) => {
setItems(items.filter(item => item.id !== id));
};

const toggleRequired = (id) => {
setItems(items.map(item =>
item.id === id ? {...item, required: !item.required} : item
));
};

const updateWeatherItems = () => {
if (weather === 'rainy' && !items.some(item => item.name === "傘")) {
setItems([...items, { name: "傘", id: 'umbrella', emoji: "☔", required: true }]);
}
};

const startChecking = () => {
updateWeatherItems();
setIsStarted(true);
setMessage(`${items[currentStep].emoji} ${items[currentStep].name}を確認してください`);
};

const handleResponse = (isYes) => {
const currentItem = items[currentStep];
setMessage(isYes ?
`${currentItem.emoji} OK！` :
`${currentItem.emoji} 忘れずに持ちましょう！`);

    setTimeout(() => {
      if (currentStep < items.length - 1) {
        setCurrentStep(prev => prev + 1);
        const nextItem = items[currentStep + 1];
        setMessage(`${nextItem.emoji} ${nextItem.name}を確認してください`);
      } else {
        setMessage('✨ チェック完了！良い一日を！');
        setTimeout(() => {
          setIsStarted(false);
          setCurrentStep(0);
          setMessage('');
        }, 2000);
      }
    }, 1000);

};

return (
<main className="min-h-screen bg-gray-100 py-6 flex flex-col justify-center sm:py-12">
<div className="relative py-3 sm:max-w-xl sm:mx-auto w-full px-4">
<div className="relative bg-white shadow-lg sm:rounded-3xl p-6 sm:p-20">
<div className="max-w-md mx-auto">
<div className="divide-y divide-gray-200">
<div className="py-8 text-base leading-6 space-y-4 text-gray-700 sm:text-lg sm:leading-7">
<div className="flex justify-between items-center">
<h1 className="text-2xl font-bold">
クイック持ち物チェッカー
<span className="text-3xl ml-2">🎒</span>
</h1>
{!isStarted && (
<button
onClick={() => setIsEditing(!isEditing)}
className="text-blue-500 hover:text-blue-600 px-4 py-2" >
✏️
</button>
)}
</div>

                {!isStarted && !isEditing && (
                  <div className="flex gap-2 mb-4">
                    <button
                      className={`p-2 rounded-full ${weather === 'sunny' ? 'bg-yellow-100' : 'bg-gray-100'}`}
                      onClick={() => setWeather('sunny')}
                    >
                      ☀️
                    </button>
                    <button
                      className={`p-2 rounded-full ${weather === 'cloudy' ? 'bg-gray-200' : 'bg-gray-100'}`}
                      onClick={() => setWeather('cloudy')}
                    >
                      ☁️
                    </button>
                    <button
                      className={`p-2 rounded-full ${weather === 'rainy' ? 'bg-blue-100' : 'bg-gray-100'}`}
                      onClick={() => setWeather('rainy')}
                    >
                      ☔
                    </button>
                  </div>
                )}

                {isEditing && (
                  <div className="space-y-4">
                    <div className="flex gap-2">
                      <input
                        type="text"
                        value={newItemName}
                        onChange={(e) => setNewItemName(e.target.value)}
                        placeholder="新しい項目を追加"
                        className="flex-1 p-2 border rounded-lg"
                      />
                      <button
                        onClick={addItem}
                        className="bg-green-500 text-white p-2 rounded-lg hover:bg-green-600"
                      >
                        ➕
                      </button>
                    </div>
                    <ul className="space-y-2">
                      {items.map(item => (
                        <li key={item.id} className="flex items-center gap-2 p-2 bg-gray-50 rounded-lg">
                          <span>{item.emoji}</span>
                          <span className="flex-1">{item.name}</span>
                          <button
                            onClick={() => toggleRequired(item.id)}
                            className={`px-2 py-1 rounded ${
                              item.required ? 'bg-blue-100 text-blue-600' : 'bg-gray-100'
                            }`}
                          >
                            必須
                          </button>
                          <button
                            onClick={() => removeItem(item.id)}
                            className="text-red-500 hover:text-red-600 px-2"
                          >
                            ❌
                          </button>
                        </li>
                      ))}
                    </ul>
                  </div>
                )}

                {message && (
                  <div className="text-center p-4 bg-blue-50 rounded-lg animate-fade-in">
                    {message}
                  </div>
                )}

                {!isStarted && !isEditing && (
                  <button
                    className="w-full bg-blue-500 text-white px-4 py-2 rounded-lg hover:bg-blue-600 transition-colors"
                    onClick={startChecking}
                  >
                    チェック開始 ▶️
                  </button>
                )}

                {isStarted && (
                  <div className="space-y-4">
                    <div className="p-4 bg-blue-50 rounded-lg text-center">
                      <p className="text-xl mb-2">{items[currentStep].emoji}</p>
                      <p className="text-lg">{items[currentStep].name}はありますか？</p>
                    </div>

                    <div className="flex gap-2">
                      <button
                        className="flex-1 bg-green-500 text-white px-4 py-2 rounded-lg hover:bg-green-600 transition-colors"
                        onClick={() => handleResponse(true)}
                      >
                        はい ✅
                      </button>
                      <button
                        className="flex-1 bg-red-500 text-white px-4 py-2 rounded-lg hover:bg-red-600 transition-colors"
                        onClick={() => handleResponse(false)}
                      >
                        いいえ ❌
                      </button>
                    </div>
                  </div>
                )}
              </div>
            </div>
          </div>
        </div>
      </div>
    </main>

);
}
