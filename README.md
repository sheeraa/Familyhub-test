<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <!-- Cache-disabling meta tags to ensure latest files load -->
    <meta http-equiv="Cache-Control" content="no-cache, no-store, must-revalidate">
    <meta http-equiv="Pragma" content="no-cache">
    <meta http-equiv="Expires" content="0">
    <title>Family Schedule App</title>
    <script src="https://cdn.jsdelivr.net/npm/react@18.2.0/umd/react.production.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/react-dom@18.2.0/umd/react-dom.production.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/@babel/standalone@7.22.9/babel.min.js"></script>
    <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="min-h-screen">
    <div id="root"></div>
    <script type="text/babel">
        const { useState, useEffect } = React;

        // Error Boundary Component
        class ErrorBoundary extends React.Component {
            state = { hasError: false, error: null };

            static getDerivedStateFromError(error) {
                return { hasError: true, error };
            }

            render() {
                if (this.state.hasError) {
                    return (
                        <div className="p-4 text-red-500">
                            <h2>Something went wrong.</h2>
                            <p>{this.state.error?.message || 'Unknown error'}</p>
                        </div>
                    );
                }
                return this.props.children;
            }
        }

        // Login/Register Component
        const LoginRegister = ({ setLoggedInUser }) => {
            const [isLogin, setIsLogin] = useState(true);
            const [username, setUsername] = useState('');
            const [password, setPassword] = useState('');
            const [confirmPassword, setConfirmPassword] = useState('');
            const [error, setError] = useState('');

            const getUsers = () => {
                try {
                    return JSON.parse(localStorage.getItem('users') || '{}');
                } catch (e) {
                    console.error('Error accessing localStorage for users:', e);
                    return {};
                }
            };

            const saveUsers = (users) => {
                try {
                    localStorage.setItem('users', JSON.stringify(users));
                } catch (e) {
                    console.error('Error saving users to localStorage:', e);
                }
            };

            const handleRegister = () => {
                setError('');
                if (!username || !password || !confirmPassword) {
                    setError('All fields are required.');
                    return;
                }
                if (password !== confirmPassword) {
                    setError('Passwords do not match.');
                    return;
                }
                const users = getUsers();
                if (users[username]) {
                    setError('Username already exists.');
                    return;
                }
                users[username] = password;
                saveUsers(users);
                setLoggedInUser(username);
                setUsername('');
                setPassword('');
                setConfirmPassword('');
            };

            const handleLogin = () => {
                setError('');
                if (!username || !password) {
                    setError('Username and password are required.');
                    return;
                }
                const users = getUsers();
                if (users[username] && users[username] === password) {
                    setLoggedInUser(username);
                    setUsername('');
                    setPassword('');
                } else {
                    setError('Invalid username or password.');
                }
            };

            return (
                <div className="flex items-center justify-center min-h-screen bg-blue-100">
                    <div className="bg-white p-8 rounded-lg shadow-md w-full max-w-sm">
                        <h2 className="text-2xl font-bold text-purple-600 mb-6 text-center">
                            {isLogin ? 'Login' : 'Register'}
                        </h2>
                        {error && (
                            <p className="text-red-500 mb-4 text-center">{error}</p>
                        )}
                        <div className="space-y-4">
                            <input
                                type="text"
                                placeholder="Username"
                                value={username}
                                onChange={(e) => setUsername(e.target.value)}
                                className="w-full border border-gray-300 p-2 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
                            />
                            <input
                                type="password"
                                placeholder="Password"
                                value={password}
                                onChange={(e) => setPassword(e.target.value)}
                                className="w-full border border-gray-300 p-2 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
                            />
                            {!isLogin && (
                                <input
                                    type="password"
                                    placeholder="Confirm Password"
                                    value={confirmPassword}
                                    onChange={(e) => setConfirmPassword(e.target.value)}
                                    className="w-full border border-gray-300 p-2 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
                                />
                            )}
                            <button
                                onClick={isLogin ? handleLogin : handleRegister}
                                className="w-full bg-blue-500 text-white px-4 py-2 rounded-lg hover:bg-blue-600 transition"
                            >
                                {isLogin ? 'Login' : 'Register'}
                            </button>
                            <button
                                onClick={() => {
                                    setIsLogin(!isLogin);
                                    setError('');
                                    setUsername('');
                                    setPassword('');
                                    setConfirmPassword('');
                                }}
                                className="w-full text-blue-500 hover:underline"
                            >
                                {isLogin ? 'Need an account? Register' : 'Already have an account? Login'}
                            </button>
                        </div>
                    </div>
                </div>
            );
        };

        // Calendar Overview Component
        const CalendarOverview = ({ meals, chores, setMeals, setChores, checklist, setChecklist, checklistItems, setChecklistItems, householdItems, setHouseholdItems, currency, broadcastState }) => {
            const [currentMonth, setCurrentMonth] = useState(new Date().getMonth());
            const [currentYear, setCurrentYear] = useState(new Date().getFullYear());
            const [currentWeek, setCurrentWeek] = useState(() => {
                const today = new Date();
                const firstDayOfMonth = new Date(today.getFullYear(), today.getMonth(), 1);
                return Math.floor((today.getDate() - 1 + firstDayOfMonth.getDay()) / 7);
            });
            const [viewMode, setViewMode] = useState('monthly');
            const [addType, setAddType] = useState('meal');
            const [selectedDate, setSelectedDate] = useState('');
            const [mealName, setMealName] = useState('');
            const [ingredients, setIngredients] = useState('');
            const [isTakeaway, setIsTakeaway] = useState(false);
            const [restaurant, setRestaurant] = useState('');
            const [takeawayPrice, setTakeawayPrice] = useState('');
            const [choreTask, setChoreTask] = useState('');
            const [assignedTo, setAssignedTo] = useState('');
            const [newChecklistItem, setNewChecklistItem] = useState('');

            const currencySymbols = {
                USD: '$',
                EUR: '€',
                GBP: '£',
                AUD: '$',
                CAD: '$'
            };

            const monthNames = [
                'January', 'February', 'March', 'April', 'May', 'June',
                'July', 'August', 'September', 'October', 'November', 'December'
            ];

            const daysInMonth = (month, year) => new Date(year, month + 1, 0).getDate();
            const firstDayOfMonth = (month, year) => new Date(year, month, 1).getDay();

            const prevMonth = () => {
                setCurrentMonth(prev => {
                    const newMonth = prev === 0 ? 11 : prev - 1;
                    setCurrentYear(curr => prev === 0 ? curr - 1 : curr);
                    return newMonth;
                });
                setCurrentWeek(0);
            };

            const nextMonth = () => {
                setCurrentMonth(prev => {
                    const newMonth = prev === 11 ? 0 : prev + 1;
                    setCurrentYear(curr => prev === 11 ? curr + 1 : curr);
                    return newMonth;
                });
                setCurrentWeek(0);
            };

            const prevWeek = () => {
                if (currentWeek === 0) {
                    prevMonth();
                    setCurrentWeek(getWeeksInMonth(currentMonth === 0 ? 11 : currentMonth - 1, currentMonth === 0 ? currentYear - 1 : currentYear) - 1);
                } else {
                    setCurrentWeek(prev => prev - 1);
                }
            };

            const nextWeek = () => {
                const weeksInMonth = getWeeksInMonth(currentMonth, currentYear);
                if (currentWeek >= weeksInMonth - 1) {
                    nextMonth();
                    setCurrentWeek(0);
                } else {
                    setCurrentWeek(prev => prev + 1);
                }
            };

            const getWeeksInMonth = (month, year) => {
                const days = daysInMonth(month, year);
                const firstDay = firstDayOfMonth(month, year);
                return Math.ceil((days + firstDay) / 7);
            };

            const getWeekDateRange = () => {
                const firstDay = firstDayOfMonth(currentMonth, currentYear);
                const daysInCurrentMonth = daysInMonth(currentMonth, currentYear);
                const startDay = currentWeek * 7 - firstDay + 1;
                const startDate = new Date(currentYear, currentMonth, Math.max(1, startDay));
                const endDate = new Date(currentYear, currentMonth, Math.min(startDay + 6, daysInCurrentMonth));
                return `${monthNames[startDate.getMonth()]} ${startDate.getDate()} - ${monthNames[endDate.getMonth()]} ${endDate.getDate()}, ${currentYear}`;
            };

            const handleAdd = () => {
                if (addType === 'meal' && selectedDate && mealName && (isTakeaway ? (restaurant && takeawayPrice) : ingredients)) {
                    setMeals(prev => {
                        const newMeals = {
                            ...prev,
                            [selectedDate]: {
                                name: mealName,
                                isTakeaway,
                                ...(isTakeaway ? { restaurant, price: parseFloat(takeawayPrice) || 0 } : { ingredients: ingredients.split(',').map(i => i.trim()) })
                            }
                        };
                        broadcastState({ meals: newMeals }); // Sync meals
                        return newMeals;
                    });
                    setMealName('');
                    setIngredients('');
                    setRestaurant('');
                    setTakeawayPrice('');
                    setIsTakeaway(false);
                } else if (addType === 'chore' && selectedDate && choreTask && assignedTo) {
                    setChores(prev => {
                        const newChores = [
                            ...prev,
                            {
                                id: Date.now(),
                                date: selectedDate,
                                task: choreTask,
                                assignedTo
                            }
                        ];
                        broadcastState({ chores: newChores }); // Sync chores
                        return newChores;
                    });
                    setChoreTask('');
                    setAssignedTo('');
                }
                setSelectedDate('');
            };

            const handleAddChecklistItem = () => {
                if (newChecklistItem && !checklistItems.includes(newChecklistItem)) {
                    setChecklistItems(prev => {
                        const newItems = [...prev, newChecklistItem];
                        broadcastState({ checklistItems: newItems }); // Sync checklist items
                        return newItems;
                    });
                    setNewChecklistItem('');
                }
            };

            const handleRemoveChecklistItem = (item) => {
                setChecklistItems(prev => {
                    const newItems = prev.filter(i => i !== item);
                    broadcastState({ checklistItems: newItems }); // Sync checklist items
                    return newItems;
                });
                setChecklist(prev => {
                    const newChecklist = { ...prev };
                    delete newChecklist[item];
                    broadcastState({ checklist: newChecklist }); // Sync checklist
                    return newChecklist;
                });
            };

            const handleChecklistChange = (itemName) => {
                setChecklist(prev => {
                    const newChecklist = { ...prev, week: prev.week || Math.floor((new Date() - new Date(new Date().getFullYear(), 0, 1)) / (1000 * 60 * 60 * 24 * 7)) };
                    newChecklist[itemName] = !prev[itemName];
                    if (newChecklist[itemName]) {
                        if (!householdItems.some(item => item.name === itemName)) {
                            setHouseholdItems(prevItems => {
                                const newItems = [
                                    ...prevItems,
                                    { id: Date.now(), name: itemName, quantity: 1, price: 0 }
                                ];
                                broadcastState({ householdItems: newItems }); // Sync household items
                                return newItems;
                            });
                        }
                    }
                    broadcastState({ checklist: newChecklist }); // Sync checklist
                    return newChecklist;
                });
            };

            const renderMonthlyCalendar = () => {
                const days = daysInMonth(currentMonth, currentYear);
                const firstDay = firstDayOfMonth(currentMonth, currentYear);
                const today = new Date();
                const isCurrentMonth = today.getMonth() === currentMonth && today.getFullYear() === currentYear;

                let cells = [];
                for (let i = 0; i < firstDay; i++) {
                    cells.push(<div key={`empty-${i}`} className="border border-gray-200"></div>);
                }

                for (let day = 1; day <= days; day++) {
                    const dateStr = `${currentYear}-${String(currentMonth + 1).padStart(2, '0')}-${String(day).padStart(2, '0')}`;
                    const meal = meals[dateStr];
                    const dayChores = chores.filter(chore => chore.date === dateStr);
                    const isToday = isCurrentMonth && day === today.getDate();
                    cells.push(
                        <div
                            key={day}
                            className={`border border-gray-200 p-4 h-56 overflow-auto ${isToday ? 'bg-blue-200' : (meal || dayChores.length) ? 'bg-gray-200' : 'bg-white'}`}
                        >
                            <div className="text-lg font-bold">{day}</div>
                            {meal && (
                                <div className="text-base bg-yellow-200 p-3 mb-3 rounded shadow-sm">
                                    <p><strong>Meal:</strong> {meal.name}</p>
                                    {meal.isTakeaway && <p className="text-gray-600">{currencySymbols[currency]}{(meal.price || 0).toFixed(2)}</p>}
                                </div>
                            )}
                            {dayChores.map(chore => (
                                <div key={chore.id} className="text-base bg-orange-200 p-3 mb-3 rounded shadow-sm">
                                    <p><strong>Chore:</strong> {chore.task}</p>
                                    <p className="text-gray-600">{chore.assignedTo}</p>
                                </div>
                            ))}
                        </div>
                    );
                }

                return cells;
            };

            const renderWeeklyCalendar = () => {
                const firstDay = firstDayOfMonth(currentMonth, currentYear);
                const daysInCurrentMonth = daysInMonth(currentMonth, currentYear);
                const startDay = currentWeek * 7 - firstDay + 1;
                const today = new Date();
                const isCurrentMonth = today.getMonth() === currentMonth && today.getFullYear() === currentYear;
                const dayNames = ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'];

                let cells = [];
                for (let i = 0; i < 7; i++) {
                    const day = startDay + i;
                    if (day < 1 || day > daysInCurrentMonth) {
                        cells.push(
                            <div
                                key={`empty-${i}`}
                                className="border border-gray-200 p-6 h-96 w-full bg-white rounded-lg shadow-sm mb-4"
                            >
                                <div className="text-xl font-bold">-</div>
                            </div>
                        );
                    } else {
                        const dateStr = `${currentYear}-${String(currentMonth + 1).padStart(2, '0')}-${String(day).padStart(2, '0')}`;
                        const meal = meals[dateStr];
                        const dayChores = chores.filter(chore => chore.date === dateStr);
                        const isToday = isCurrentMonth && day === today.getDate();
                        cells.push(
                            <div
                                key={day}
                                className={`border border-gray-200 p-6 h-96 w-full rounded-lg shadow-sm mb-4 ${
                                    isToday ? 'bg-blue-200' : (meal || dayChores.length) ? 'bg-gray-200' : 'bg-white'
                                }`}
                            >
                                <div className="text-xl font-bold mb-4">{day} {dayNames[i]}</div>
                                <div className="space-y-4">
                                    <div>
                                        <h3 className="text-lg font-semibold text-yellow-800">Meal</h3>
                                        <div className="bg-yellow-200 p-4 rounded shadow-sm overflow-auto max-h-32">
                                            {meal ? (
                                                <>
                                                    <p className="text-base"><strong>{meal.name}</strong></p>
                                                    {meal.isTakeaway && (
                                                        <p className="text-gray-600">{currencySymbols[currency]}{(meal.price || 0).toFixed(2)}</p>
                                                    )}
                                                </>
                                            ) : (
                                                <p className="text-base text-gray-600">No meal planned</p>
                                            )}
                                        </div>
                                    </div>
                                    <div>
                                        <h3 className="text-lg font-semibold text-orange-800">Chores</h3>
                                        <div className="bg-orange-200 p-4 rounded shadow-sm overflow-auto max-h-32">
                                            {dayChores.length > 0 ? (
                                                dayChores.map(chore => (
                                                    <div key={chore.id} className="mb-2 last:mb-0">
                                                        <p className="text-base"><strong>{chore.task}</strong></p>
                                                        <p className="text-gray-600">{chore.assignedTo}</p>
                                                    </div>
                                                ))
                                            ) : (
                                                <p className="text-base text-gray-600">No chores assigned</p>
                                            )}
                                        </div>
                                    </div>
                                </div>
                            </div>
                        );
                    }
                }

                return cells;
            };

            return (
                <div className="space-y-4">
                    <div className="p-4 bg-white rounded-lg shadow-md">
                        <div className="flex items-center justify-between mb-4">
                            {viewMode === 'monthly' ? (
                                <>
                                    <button onClick={prevMonth} className="bg-blue-500 text-white px-4 py-2 rounded-lg hover:bg-blue-600">
                                        Previous
                                    </button>
                                    <h2 className="text-2xl font-bold text-purple-600">{monthNames[currentMonth]} {currentYear}</h2>
                                    <button onClick={nextMonth} className="bg-blue-500 text-white px-4 py-2 rounded-lg hover:bg-blue-600">
                                        Next
                                    </button>
                                </>
                            ) : (
                                <>
                                    <button onClick={prevWeek} className="bg-blue-500 text-white px-4 py-2 rounded-lg hover:bg-blue-600">
                                        Previous Week
                                    </button>
                                    <h2 className="text-2xl font-bold text-purple-600">Week of {getWeekDateRange()}</h2>
                                    <button onClick={nextWeek} className="bg-blue-500 text-white px-4 py-2 rounded-lg hover:bg-blue-600">
                                        Next Week
                                    </button>
                                </>
                            )}
                        </div>
                        <div className="flex justify-center mb-4">
                            <button
                                onClick={() => setViewMode(viewMode === 'monthly' ? 'weekly' : 'monthly')}
                                className="bg-purple-500 text-white px-4 py-2 rounded-lg hover:bg-purple-600 transition"
                            >
                                Switch to {viewMode === 'monthly' ? 'Weekly' : 'Monthly'} View
                            </button>
                        </div>
                        {viewMode === 'monthly' ? (
                            <div className="grid grid-cols-7 gap-1 text-center overflow-x-auto">
                                {['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'].map(day => (
                                    <div key={day} className="font-semibold text-gray-700 border border-gray-200 p-4 text-base">{day}</div>
                                ))}
                                {renderMonthlyCalendar()}
                            </div>
                        ) : (
                            <div className="flex flex-col gap-4">
                                {renderWeeklyCalendar()}
                            </div>
                        )}
                    </div>
                    <div className="p-4 bg-white rounded-lg shadow-md">
                        <div className="flex items-center mb-4">
                            <svg className="w-8 h-8 text-pink-600 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                <path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z" />
                            </svg>
                            <h2 className="text-2xl font-bold text-purple-600">Add Meal or Chore</h2>
                        </div>
                        <div className="flex flex-col gap-4">
                            <select
                                value={addType}
                                onChange={(e) => setAddType(e.target.value)}
                                className="border border-gray-300 p-2 rounded-lg"
                            >
                                <option value="meal">Meal</option>
                                <option value="chore">Chore</option>
                            </select>
                            <input
                                type="date"
                                value={selectedDate}
                                onChange={(e) => setSelectedDate(e.target.value)}
                                className="border border-gray-300 p-2 rounded-lg"
                            />
                            {addType === 'meal' ? (
                                <>
                                    <input
                                        type="text"
                                        placeholder="Meal Name"
                                        value={mealName}
                                        onChange={(e) => setMealName(e.target.value)}
                                        className="border border-gray-300 p-2 rounded-lg"
                                    />
                                    <div className="flex items-center gap-2">
                                        <input
                                            type="checkbox"
                                            checked={isTakeaway}
                                            onChange={() => setIsTakeaway(!isTakeaway)}
                                            className="w-5 h-5 text-blue-600"
                                        />
                                        <label className="text-gray-700">Takeaway</label>
                                    </div>
                                    {isTakeaway ? (
                                        <>
                                            <input
                                                type="text"
                                                placeholder="Restaurant Name"
                                                value={restaurant}
                                                onChange={(e) => setRestaurant(e.target.value)}
                                                className="border border-gray-300 p-2 rounded-lg"
                                            />
                                            <input
                                                type="number"
                                                placeholder={`Price (${currency})`}
                                                value={takeawayPrice}
                                                onChange={(e) => setTakeawayPrice(e.target.value)}
                                                className="border border-gray-300 p-2 rounded-lg"
                                                step="0.01"
                                                min="0"
                                            />
                                        </>
                                    ) : (
                                        <input
                                            type="text"
                                            placeholder="Ingredients (comma-separated)"
                                            value={ingredients}
                                            onChange={(e) => setIngredients(e.target.value)}
                                            className="border border-gray-300 p-2 rounded-lg"
                                        />
                                    )}
                                </>
                            ) : (
                                <>
                                    <input
                                        type="text"
                                        placeholder="Chore Task"
                                        value={choreTask}
                                        onChange={(e) => setChoreTask(e.target.value)}
                                        className="border border-gray-300 p-2 rounded-lg"
                                    />
                                    <input
                                        type="text"
                                        placeholder="Assigned To"
                                        value={assignedTo}
                                        onChange={(e) => setAssignedTo(e.target.value)}
                                        className="border border-gray-300 p-2 rounded-lg"
                                    />
                                </>
                            )}
                            <button
                                onClick={handleAdd}
                                className="bg-blue-500 text-white px-4 py-2 rounded-lg hover:bg-blue-600 transition"
                            >
                                Add
                            </button>
                        </div>
                    </div>
                    <div className="p-4 bg-white rounded-lg shadow-md">
                        <div className="flex items-center mb-4">
                            <svg className="w-8 h-8 text-green-600 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                <path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z" />
                            </svg>
                            <h2 className="text-2xl font-bold text-purple-600">Weekly Household Checklist</h2>
                        </div>
                        <div className="flex gap-2 mb-4">
                            <input
                                type="text"
                                placeholder="Add Checklist Item"
                                value={newChecklistItem}
                                onChange={(e) => setNewChecklistItem(e.target.value)}
                                className="border border-gray-300 p-2 rounded-lg flex-grow"
                            />
                            <button
                                onClick={handleAddChecklistItem}
                                className="bg-green-500 text-white px-4 py-2 rounded-lg hover:bg-green-600 transition"
                            >
                                Add
                            </button>
                        </div>
                        {checklistItems.map(item => (
                            <div key={item} className="flex items-center gap-2 mb-1 bg-orange-50 p-2 rounded">
                                <input
                                    type="checkbox"
                                    checked={checklist[item] || false}
                                    onChange={() => handleChecklistChange(item)}
                                    className="w-5 h-5 text-blue-600"
                                />
                                <span className="text-gray-700">{item}</span>
                                <button
                                    onClick={() => handleRemoveChecklistItem(item)}
                                    className="text-red-500 hover:text-red-700"
                                >
                                    Remove
                                </button>
                            </div>
                        ))}
                    </div>
                </div>
            );
        };

        // Meals Section Component
        const MealsSection = ({ setMeals, currency, meals, broadcastState }) => {
            const [selectedDate, setSelectedDate] = useState('');
            const [mealName, setMealName] = useState('');
            const [ingredients, setIngredients] = useState('');
            const [isTakeaway, setIsTakeaway] = useState(false);
            const [restaurant, setRestaurant] = useState('');
            const [takeawayPrice, setTakeawayPrice] = useState('');
            const [currentMonth, setCurrentMonth] = useState(new Date().getMonth());
            const [currentYear, setCurrentYear] = useState(new Date().getFullYear());

            const currencySymbols = {
                USD: '$',
                EUR: '€',
                GBP: '£',
                AUD: '$',
                CAD: '$'
            };

            const monthNames = [
                'January', 'February', 'March', 'April', 'May', 'June',
                'July', 'August', 'September', 'October', 'November', 'December'
            ];

            const prevMonth = () => {
                setCurrentMonth(prev => {
                    const newMonth = prev === 0 ? 11 : prev - 1;
                    setCurrentYear(curr => prev === 0 ? curr - 1 : curr);
                    return newMonth;
                });
            };

            const nextMonth = () => {
                setCurrentMonth(prev => {
                    const newMonth = prev === 11 ? 0 : prev + 1;
                    setCurrentYear(curr => prev === 11 ? curr + 1 : curr);
                    return newMonth;
                });
            };

            const handleAddMeal = () => {
                if (selectedDate && mealName && (isTakeaway ? (restaurant && takeawayPrice) : ingredients)) {
                    setMeals(prev => {
                        const newMeals = {
                            ...prev,
                            [selectedDate]: {
                                name: mealName,
                                isTakeaway,
                                ...(isTakeaway ? { restaurant, price: parseFloat(takeawayPrice) || 0 } : { ingredients: ingredients.split(',').map(i => i.trim()) })
                            }
                        };
                        broadcastState({ meals: newMeals }); // Sync meals
                        return newMeals;
                    });
                    setMealName('');
                    setIngredients('');
                    setRestaurant('');
                    setTakeawayPrice('');
                    setIsTakeaway(false);
                    setSelectedDate('');
                }
            };

            const removeMeal = (date) => {
                setMeals(prev => {
                    const newMeals = { ...prev };
                    delete newMeals[date];
                    broadcastState({ meals: newMeals }); // Sync meals
                    return newMeals;
                });
            };

            return (
                <div className="space-y-4">
                    <div className="p-4 bg-white rounded-lg shadow-md">
                        <div className="flex items-center mb-4">
                            <svg className="w-8 h-8 text-pink-600 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                <path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z" />
                            </svg>
                            <h2 className="text-2xl font-bold text-purple-600">Add Meal</h2>
                        </div>
                        <div className="flex flex-col gap-4">
                            <input
                                type="date"
                                value={selectedDate}
                                onChange={(e) => setSelectedDate(e.target.value)}
                                className="border border-gray-300 p-2 rounded-lg"
                            />
                            <input
                                type="text"
                                placeholder="Meal Name"
                                value={mealName}
                                onChange={(e) => setMealName(e.target.value)}
                                className="border border-gray-300 p-2 rounded-lg"
                            />
                            <div className="flex items-center gap-2">
                                <input
                                    type="checkbox"
                                    checked={isTakeaway}
                                    onChange={() => setIsTakeaway(!isTakeaway)}
                                    className="w-5 h-5 text-blue-600"
                                />
                                <label className="text-gray-700">Takeaway</label>
                            </div>
                            {isTakeaway ? (
                                <>
                                    <input
                                        type="text"
                                        placeholder="Restaurant Name"
                                        value={restaurant}
                                        onChange={(e) => setRestaurant(e.target.value)}
                                        className="border border-gray-300 p-2 rounded-lg"
                                    />
                                    <input
                                        type="number"
                                        placeholder={`Price (${currency})`}
                                        value={takeawayPrice}
                                        onChange={(e) => setTakeawayPrice(e.target.value)}
                                        className="border border-gray-300 p-2 rounded-lg"
                                        step="0.01"
                                        min="0"
                                    />
                                </>
                            ) : (
                                <input
                                    type="text"
                                    placeholder="Ingredients (comma-separated)"
                                    value={ingredients}
                                    onChange={(e) => setIngredients(e.target.value)}
                                    className="border border-gray-300 p-2 rounded-lg"
                                />
                            )}
                            <button
                                onClick={handleAddMeal}
                                className="bg-blue-500 text-white px-4 py-2 rounded-lg hover:bg-blue-600 transition"
                            >
                                Add Meal
                            </button>
                        </div>
                    </div>
                    <div className="p-4 bg-white rounded-lg shadow-md">
                        <div className="flex items-center justify-between mb-4">
                            <button onClick={prevMonth} className="bg-blue-500 text-white px-3 py-1 rounded-lg hover:bg-blue-600">
                                Previous
                            </button>
                            <h2 className="text-2xl font-bold text-purple-600">Monthly Meal Plan - {monthNames[currentMonth]} {currentYear}</h2>
                            <button onClick={nextMonth} className="bg-blue-500 text-white px-3 py-1 rounded-lg hover:bg-blue-600">
                                Next
                            </button>
                        </div>
                        <div className="space-y-4">
                            {Object.entries(meals || {})
                                .filter(([date]) => {
                                    const mealDate = new Date(date);
                                    return mealDate.getFullYear() === currentYear && mealDate.getMonth() === currentMonth;
                                })
                                .map(([date, meal]) => (
                                    <div key={date} className="border border-gray-200 p-4 rounded-lg bg-yellow-50">
                                        <p><strong>Date:</strong> {date}</p>
                                        <p><strong>Meal:</strong> {meal.name}</p>
                                        {meal.isTakeaway ? (
                                            <>
                                                <p><strong>Restaurant:</strong> {meal.restaurant}</p>
                                                <p><strong>Price:</strong> {currencySymbols[currency]}{(meal.price || 0).toFixed(2)}</p>
                                            </>
                                        ) : (
                                            <p><strong>Ingredients:</strong> {meal.ingredients?.join(', ') || ''}</p>
                                        )}
                                        <button
                                            onClick={() => removeMeal(date)}
                                            className="mt-2 bg-red-500 text-white px-4 py-2 rounded-lg hover:bg-red-600 transition"
                                        >
                                            Remove
                                        </button>
                                    </div>
                                ))}
                        </div>
                    </div>
                </div>
            );
        };

        // Shopping List Component
        const ShoppingList = ({ meals, ingredientPrices, setIngredientPrices, householdItems, setHouseholdItems, currency, broadcastState }) => {
            const [itemName, setItemName] = useState('');
            const [itemQuantity, setItemQuantity] = useState('');
            const [itemPrice, setItemPrice] = useState('');
            const [ingredientQuantities, setIngredientQuantities] = useState(() => {
                try {
                    return JSON.parse(localStorage.getItem('ingredientQuantities') || '{}');
                } catch (e) {
                    console.error('Error accessing localStorage for ingredientQuantities:', e);
                    return {};
                }
            });
            const [removedIngredients, setRemovedIngredients] = useState(() => {
                try {
                    return JSON.parse(localStorage.getItem('removedIngredients') || '[]');
                } catch (e) {
                    console.error('Error accessing localStorage for removedIngredients:', e);
                    return [];
                }
            });

            const currencySymbols = {
                USD: '$',
                EUR: '€',
                GBP: '£',
                AUD: '$',
                CAD: '$'
            };

            useEffect(() => {
                try {
                    localStorage.setItem('ingredientQuantities', JSON.stringify(ingredientQuantities));
                    broadcastState({ ingredientQuantities }); // Sync ingredient quantities
                } catch (e) {
                    console.error('Error saving ingredientQuantities to localStorage:', e);
                }
            }, [ingredientQuantities]);

            useEffect(() => {
                try {
                    localStorage.setItem('removedIngredients', JSON.stringify(removedIngredients));
                    broadcastState({ removedIngredients }); // Sync removed ingredients
                } catch (e) {
                    console.error('Error saving removedIngredients to localStorage:', e);
                }
            }, [removedIngredients]);

            const generateShoppingList = () => {
                const ingredients = Object.values(meals).reduce((acc, meal) => {
                    if (!meal.isTakeaway && meal.ingredients) {
                        meal.ingredients.forEach(ing => {
                            if (!removedIngredients.includes(ing)) {
                                acc[ing] = (acc[ing] || { count: 0 });
                                acc[ing].count += 1;
                            }
                        });
                    }
                    return acc;
                }, {});
                Object.keys(ingredients).forEach(ing => {
                    if (ingredientQuantities[ing]) {
                        ingredients[ing].count = ingredientQuantities[ing];
                    }
                });
                return ingredients;
            };

            const increaseIngredientQuantity = (ingredient) => {
                setIngredientQuantities(prev => {
                    const newQuantities = {
                        ...prev,
                        [ingredient]: (prev[ingredient] || generateShoppingList()[ingredient]?.count || 1) + 1
                    };
                    broadcastState({ ingredientQuantities: newQuantities }); // Sync
                    return newQuantities;
                });
            };

            const decreaseIngredientQuantity = (ingredient) => {
                setIngredientQuantities(prev => {
                    const currentCount = prev[ingredient] || generateShoppingList()[ingredient]?.count || 1;
                    if (currentCount <= 1) return prev;
                    const newQuantities = {
                        ...prev,
                        [ingredient]: currentCount - 1
                    };
                    broadcastState({ ingredientQuantities: newQuantities }); // Sync
                    return newQuantities;
                });
            };

            const removeIngredient = (ingredient) => {
                setRemovedIngredients(prev => {
                    const newRemoved = [...prev, ingredient];
                    broadcastState({ removedIngredients: newRemoved }); // Sync
                    return newRemoved;
                });
                setIngredientQuantities(prev => {
                    const newQuantities = { ...prev };
                    delete newQuantities[ingredient];
                    broadcastState({ ingredientQuantities: newQuantities }); // Sync
                    return newQuantities;
                });
            };

            const increaseHouseholdItemQuantity = (id) => {
                setHouseholdItems(prev => {
                    const newItems = prev.map(item =>
                        item.id === id ? { ...item, quantity: item.quantity + 1 } : item
                    );
                    broadcastState({ householdItems: newItems }); // Sync
                    return newItems;
                });
            };

            const decreaseHouseholdItemQuantity = (id) => {
                setHouseholdItems(prev => {
                    const newItems = prev.map(item =>
                        item.id === id && item.quantity > 1 ? { ...item, quantity: item.quantity - 1 } : item
                    );
                    broadcastState({ householdItems: newItems }); // Sync
                    return newItems;
                });
            };

            const handleAddHouseholdItem = () => {
                const quantity = parseInt(itemQuantity) || 1;
                if (itemName && quantity > 0 && itemPrice) {
                    setHouseholdItems(prev => {
                        const newItems = [
                            ...prev,
                            {
                                id: Date.now(),
                                name: itemName,
                                quantity,
                                price: parseFloat(itemPrice) || 0
                            }
                        ];
                        broadcastState({ householdItems: newItems }); // Sync
                        return newItems;
                    });
                    setItemName('');
                    setItemQuantity('');
                    setItemPrice('');
                }
            };

            const handleRemoveHouseholdItem = (id) => {
                setHouseholdItems(prev => {
                    const newItems = prev.filter(item => item.id !== id);
                    broadcastState({ householdItems: newItems }); // Sync
                    return newItems;
                });
            };

            const shoppingList = generateShoppingList();
            const totalCost = Object.entries(shoppingList).reduce((sum, [ingredient, { count }]) => {
                const price = ingredientPrices[ingredient] || 0;
                return sum + price * count;
            }, 0) + householdItems.reduce((sum, item) => sum + item.quantity * item.price, 0);

            return (
                <div className="p-4 bg-white rounded-lg shadow-md">
                    <div className="flex items-center mb-4">
                        <svg className="w-8 h-8 text-green-600 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M3 3h2l.4 2M7 13h10l4-8H5.4M7 13L5.4 5M7 13l-2.293 2.293c-.63.63-.184 1.707.707 1.707H17m0 0a2 2 0 100 4 2 2 0 000-4zm-8 2a2 2 0 11-4 0 2 2 0 014 0z" />
                        </svg>
                        <h2 className="text-2xl font-bold text-purple-600">Shopping List</h2>
                    </div>
                    <div className="mb-4">
                        <h3 className="text-xl font-semibold text-blue-600 mb-2">Add Household Item</h3>
                        <div className="flex flex-col gap-2">
                            <input
                                type="text"
                                placeholder="Item Name (e.g., Toilet Rolls)"
                                value={itemName}
                                onChange={(e) => setItemName(e.target.value)}
                                className="border border-gray-300 p-2 rounded-lg"
                            />
                            <input
                                type="number"
                                placeholder="Quantity"
                                value={itemQuantity}
                                onChange={(e) => setItemQuantity(e.target.value)}
                                className="border border-gray-300 p-2 rounded-lg"
                                min="1"
                            />
                            <input
                                type="number"
                                placeholder={`Price (${currency})`}
                                value={itemPrice}
                                onChange={(e) => setItemPrice(e.target.value)}
                                className="border border-gray-300 p-2 rounded-lg"
                                step="0.01"
                                min="0"
                            />
                            <button
                                onClick={handleAddHouseholdItem}
                                className="bg-blue-500 text-white px-4 py-2 rounded-lg hover:bg-blue-600 transition"
                            >
                                Add Item
                            </button>
                        </div>
                    </div>
                    <h3 className="text-xl font-semibold text-blue-600 mb-2">Meal Ingredients</h3>
                    <ul className="list-disc pl-5 mb-4">
                        {Object.entries(shoppingList).map(([ingredient, { count }]) => (
                            <li key={ingredient} className="flex items-center gap-4 mb-2">
                                <span className="text-gray-700 flex-1">{ingredient}</span>
                                <div className="flex items-center gap-2">
                                    <button
                                        onClick={() => decreaseIngredientQuantity(ingredient)}
                                        className="bg-blue-500 text-white rounded-full w-8 h-8 flex items-center justify-center hover:bg-blue-600"
                                    >
                                        -
                                    </button>
                                    <span className="text-gray-700 w-8 text-center">x{count}</span>
                                    <button
                                        onClick={() => increaseIngredientQuantity(ingredient)}
                                        className="bg-blue-500 text-white rounded-full w-8 h-8 flex items-center justify-center hover:bg-blue-600"
                                    >
                                        +
                                    </button>
                                </div>
                                <input
                                    type="number"
                                    placeholder={`Price (${currency})`}
                                    value={ingredientPrices[ingredient] || ''}
                                    onChange={(e) => {
                                        const price = parseFloat(e.target.value) || 0;
                                        setIngredientPrices(prev => {
                                            const newPrices = { ...prev, [ingredient]: price };
                                            broadcastState({ ingredientPrices: newPrices }); // Sync
                                            return newPrices;
                                        });
                                    }}
                                    className="border border-gray-300 p-1 rounded-lg w-24"
                                    step="0.01"
                                    min="0"
                                />
                                <span className="text-gray-700 w-20">{currencySymbols[currency]}{((ingredientPrices[ingredient] || 0) * count).toFixed(2)}</span>
                                <button
                                    onClick={() => removeIngredient(ingredient)}
                                    className="bg-red-500 text-white px-2 py-1 rounded-lg hover:bg-red-600 transition"
                                >
                                    Remove
                                </button>
                            </li>
                        ))}
                    </ul>
                    <h3 className="text-xl font-semibold text-blue-600 mb-2">Household Items</h3>
                    <ul className="list-disc pl-5 mb-4">
                        {householdItems.map(item => (
                            <li key={item.id} className="flex items-center gap-4 mb-2">
                                <span className="text-gray-700 flex-1">{item.name}</span>
                                <div className="flex items-center gap-2">
                                    <button
                                        onClick={() => decreaseHouseholdItemQuantity(item.id)}
                                        className="bg-blue-500 text-white rounded-full w-8 h-8 flex items-center justify-center hover:bg-blue-600"
                                    >
                                        -
                                    </button>
                                    <span className="text-gray-700 w-8 text-center">x{item.quantity}</span>
                                    <button
                                        onClick={() => increaseHouseholdItemQuantity(item.id)}
                                        className="bg-blue-500 text-white rounded-full w-8 h-8 flex items-center justify-center hover:bg-blue-600"
                                    >
                                        +
                                    </button>
                                </div>
                                <span className="text-gray-700 w-20">{currencySymbols[currency]}{(item.price * item.quantity).toFixed(2)}</span>
                                <button
                                    onClick={() => handleRemoveHouseholdItem(item.id)}
                                    className="bg-red-500 text-white px-2 py-1 rounded-lg hover:bg-red-600 transition"
                                >
                                    Remove
                                </button>
                            </li>
                        ))}
                    </ul>
                    <p className="mt-4 font-bold text-lg text-purple-600">Total Cost: {currencySymbols[currency]}{totalCost.toFixed(2)}</p>
                </div>
            );
        };

        // Chores List Component
        const ChoresList = ({ chores, setChores, broadcastState }) => {
            const removeChore = (id) => {
                setChores(prev => {
                    const newChores = prev.filter(chore => chore.id !== id);
                    broadcastState({ chores: newChores }); // Sync chores
                    return newChores;
                });
            };

            return (
                <div className="p-4 bg-white rounded-lg shadow-md">
                    <div className="flex items-center mb-4">
                        <svg className="w-8 h-8 text-orange-600 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2m-3 7h3m-3 4h3m-6-4h.01M9 16h.01" />
                        </svg>
                        <h2 className="text-2xl font-bold text-purple-600">Chores List</h2>
                    </div>
                    <div className="space-y-4">
                        {chores.map(chore => (
                            <div key={chore.id} className="border border-gray-200 p-4 rounded-lg bg-orange-50">
                                <p><strong>Date:</strong> {chore.date}</p>
                                <p><strong>Chore:</strong> {chore.task}</p>
                                <p><strong>Assigned to:</strong> {chore.assignedTo}</p>
                                <button
                                    onClick={() => removeChore(chore.id)}
                                    className="mt-2 bg-red-500 text-white px-4 py-2 rounded-lg hover:bg-red-600 transition"
                                >
                                    Remove
                                </button>
                            </div>
                        ))}
                    </div>
                </div>
            );
        };

        // Chores Form Component
        const ChoresForm = ({ setChores, broadcastState }) => {
            const [choreDate, setChoreDate] = useState('');
            const [choreTask, setChoreTask] = useState('');
            const [assignedTo, setAssignedTo] = useState('');

            const handleAddChore = () => {
                if (choreDate && choreTask && assignedTo) {
                    setChores(prev => {
                        const newChores = [
                            ...prev,
                            {
                                id: Date.now(),
                                date: choreDate,
                                task: choreTask,
                                assignedTo
                            }
                        ];
                        broadcastState({ chores: newChores }); // Sync chores
                        return newChores;
                    });
                    setChoreDate('');
                    setChoreTask('');
                    setAssignedTo('');
                }
            };

            return (
                <div className="p-4 bg-white rounded-lg shadow-md">
                    <div className="flex items-center mb-4">
                        <svg className="w-8 h-8 text-orange-600 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2m-3 7h3m-3 4h3m-6-4h.01M9 16h.01" />
                        </svg>
                        <h2 className="text-2xl font-bold text-purple-600">Add Chore</h2>
                    </div>
                    <div className="flex flex-col gap-4">
                        <input
                            type="date"
                            value={choreDate}
                            onChange={(e) => setChoreDate(e.target.value)}
                            className="border border-gray-300 p-2 rounded-lg"
                        />
                        <input
                            type="text"
                            placeholder="Chore Task"
                            value={choreTask}
                            onChange={(e) => setChoreTask(e.target.value)}
                            className="border border-gray-300 p-2 rounded-lg"
                        />
                        <input
                            type="text"
                            placeholder="Assigned To"
                            value={assignedTo}
                            onChange={(e) => setAssignedTo(e.target.value)}
                            className="border border-gray-300 p-2 rounded-lg"
                        />
                        <button
                            onClick={handleAddChore}
                            className="bg-blue-500 text-white px-4 py-2 rounded-lg hover:bg-blue-600 transition"
                        >
                            Add Chore
                        </button>
                    </div>
                </div>
            );
        };

        // Monthly Spending Component
        const MonthlySpending = ({ meals, ingredientPrices, householdItems, currency }) => {
            const currencySymbols = {
                USD: '$',
                EUR: '€',
                GBP: '£',
                AUD: '$',
                CAD: '$'
            };

            const [monthlyCosts, setMonthlyCosts] = useState(() => {
                try {
                    return JSON.parse(localStorage.getItem('monthlyCosts') || '[]');
                } catch (e) {
                    console.error('Error accessing localStorage for monthlyCosts:', e);
                    return [];
                }
            });

            useEffect(() => {
                try {
                    localStorage.setItem('monthlyCosts', JSON.stringify(monthlyCosts));
                } catch (e) {
                    console.error('Error saving monthlyCosts to localStorage:', e);
                }
            }, [monthlyCosts]);

            const getMonthlyTotal = (year, month) => {
                const monthlyMeals = Object.entries(meals).filter(([date]) => {
                    const mealDate = new Date(date);
                    return mealDate.getFullYear() === year && mealDate.getMonth() === month;
                });

                let total = 0;
                const ingredients = monthlyMeals.reduce((acc, [, meal]) => {
                    if (!meal.isTakeaway && meal.ingredients) {
                        meal.ingredients.forEach(ing => {
                            acc[ing] = (acc[ing] || 0) + 1;
                        });
                    } else if (typeof meal.price === 'number') {
                        total += meal.price;
                    }
                    return acc;
                }, {});

                total += Object.entries(ingredients).reduce((sum, [ingredient, count]) => {
                    const price = ingredientPrices[ingredient] || 0;
                    return sum + price * count;
                }, 0);

                total += householdItems.reduce((sum, item) => sum + item.quantity * item.price, 0);

                return total;
            };

            const savePreviousMonth = () => {
                const now = new Date();
                const currentYear = now.getFullYear();
                const currentMonth = now.getMonth();
                const lastSaved = localStorage.getItem('lastSavedMonth');
                const lastSavedParsed = lastSaved ? JSON.parse(lastSaved) : null;

                if (!lastSavedParsed || lastSavedParsed.year !== currentYear || lastSavedParsed.month !== currentMonth) {
                    const prevMonth = currentMonth === 0 ? 11 : currentMonth - 1;
                    const prevYear = currentMonth === 0 ? currentYear - 1 : currentYear;
                    const prevTotal = getMonthlyTotal(prevYear, prevMonth);

                    if (prevTotal > 0) {
                        setMonthlyCosts(prev => [
                            ...prev,
                            { id: `${prevYear}-${prevMonth}`, year: prevYear, month: prevMonth, total: prevTotal }
                        ]);
                    }

                    localStorage.setItem('lastSavedMonth', JSON.stringify({ year: currentYear, month: currentMonth }));
                }
            };

            useEffect(() => {
                savePreviousMonth();
            }, [meals, ingredientPrices, householdItems]);

            const currentYear = new Date().getFullYear();
            const currentMonth = new Date().getMonth();
            const monthlyTotal = getMonthlyTotal(currentYear, currentMonth);

            const monthNames = [
                'January', 'February', 'March', 'April', 'May', 'June',
                'July', 'August', 'September', 'October', 'November', 'December'
            ];

            return (
                <div className="p-4 bg-white rounded-lg shadow-md">
                    <div className="flex items-center mb-4">
                        <svg className="w-8 h-8 text-yellow-600 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M17 9V7a2 2 0 00-2-2H5a2 2 0 00-2 2v6a2 2 0 002 2h2m2 4h10a2 2 0 002-2v-6a2 2 0 00-2-2H9a2 2 0 00-2 2v6a2 2 0 002 2zm7-5a2 2 0 11-4 0 2 2 0 014 0z" />
                        </svg>
                        <h2 className="text-2xl font-bold text-purple-600">Monthly Spending</h2>
                    </div>
                    <p className="text-lg text-gray-700 mb-4">
                        Total spent this month ({monthNames[currentMonth]} {currentYear}): <span className="font-bold text-purple-600">{currencySymbols[currency]}{monthlyTotal.toFixed(2)}</span>
                    </p>
                    <h3 className="text-xl font-semibold text-blue-600 mb-2">Previous Months</h3>
                    <div className="space-y-2">
                        {monthlyCosts.length > 0 ? (
                            monthlyCosts.map(cost => (
                                <div key={cost.id} className="border border-gray-200 p-2 rounded-lg bg-yellow-50">
                                    <p className="text-gray-700">{monthNames[cost.month]} {cost.year}: <span className="font-bold">{currencySymbols[currency]}{cost.total.toFixed(2)}</span></p>
                                </div>
                            ))
                        ) : (
                            <p className="text-gray-700">No previous monthly costs recorded.</p>
                        )}
                    </div>
                </div>
            );
        };

        // Settings Component
        const Settings = ({ currency, setCurrency, backgroundImage, setBackgroundImage, handleLogout, broadcastState }) => {
            const handleImageUpload = (e) => {
                const file = e.target.files[0];
                if (file) {
                    const reader = new FileReader();
                    reader.onload = () => {
                        setBackgroundImage(reader.result);
                        broadcastState({ backgroundImage: reader.result }); // Sync background image
                    };
                    reader.readAsDataURL(file);
                }
            };

            const handleCurrencyChange = (e) => {
                const newCurrency = e.target.value;
                setCurrency(newCurrency);
                broadcastState({ currency: newCurrency }); // Sync currency
            };

            return (
                <div className="p-4 bg-white rounded-lg shadow-md">
                    <div className="flex items-center mb-4">
                        <svg className="w-8 h-8 text-gray-600 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.065 2.572c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.572 1.065c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.065-2.572c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.296.07 2.572-1.065z" />
                            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z" />
                        </svg>
                        <h2 className="text-2xl font-bold text-purple-600">Settings</h2>
                    </div>
                    <div className="space-y-4">
                        <div>
                            <label className="text-gray-700 font-semibold mb-2 block">Currency</label>
                            <select
                                value={currency}
                                onChange={handleCurrencyChange}
                                className="border border-gray-300 p-2 rounded-lg w-full"
                            >
                                <option value="USD">USD ($)</option>
                                <option value="EUR">EUR (€)</option>
                                <option value="GBP">GBP (£)</option>
                                <option value="AUD">AUD ($)</option>
                                <option value="CAD">CAD ($)</option>
                            </select>
                        </div>
                        <div>
                            <label className="text-gray-700 font-semibold mb-2 block">Background Image</label>
                            <input
                                type="file"
                                accept="image/*"
                                onChange={handleImageUpload}
                                className="border border-gray-300 p-2 rounded-lg w-full"
                            />
                            {backgroundImage && (
                                <div className="mt-2">
                                    <img src={backgroundImage} alt="Background Preview" className="w-32 h-32 object-cover rounded" />
                                </div>
                            )}
                        </div>
                        <button
                            onClick={handleLogout}
                            className="w-full bg-red-500 text-white px-4 py-2 rounded-lg hover:bg-red-600 transition"
                        >
                            Logout
                        </button>
                    </div>
                </div>
            );
        };

        // Navigation Component
        const Navigation = ({ setCurrentSection, currentSection }) => {
            const sections = [
                { id: 'calendar', label: 'Calendar' },
                { id: 'meals', label: 'Meals' },
                { id: 'shopping', label: 'Shopping' },
                { id: 'chores', label: 'Chores' },
                { id: 'spending', label: 'Spending' },
                { id: 'settings', label: 'Settings' }
            ];

            return (
                <nav className="bg-purple-600 p-4 shadow-md">
                    <ul className="flex flex-wrap gap-4 justify-center">
                        {sections.map(section => (
                            <li key={section.id}>
                                <button
                                    onClick={() => setCurrentSection(section.id)}
                                    className={`text-white px-4 py-2 rounded-lg transition ${
                                        currentSection === section.id ? 'bg-purple-800' : 'hover:bg-purple-700'
                                    }`}
                                >
                                    {section.label}
                                </button>
                            </li>
                        ))}
                    </ul>
                </nav>
            );
        };

        const App = () => {
    const [loggedInUser, setLoggedInUser] = useState(null);
    const [currentSection, setCurrentSection] = useState('calendar');
    const [backgroundImage, setBackgroundImage] = useState('');
    const [meals, setMeals] = useState({});
    const [chores, setChores] = useState([]);
    const [ingredientPrices, setIngredientPrices] = useState({});
    const [householdItems, setHouseholdItems] = useState([]);
    const [checklist, setChecklist] = useState({
        week: Math.floor((new Date() - new Date(new Date().getFullYear(), 0, 1)) / (1000 * 60 * 60 * 24 * 7))
    });
    const [checklistItems, setChecklistItems] = useState([]);
    const [currency, setCurrency] = useState('USD');
    const [ws, setWs] = useState(null);

    // Initialize WebSocket connection
    useEffect(() => {
        const websocket = new WebSocket('ws://192.168.242.16:8080'); // Replace with your PC’s IP
        websocket.onopen = () => {
            console.log('Connected to WebSocket server');
            // Send initial state if user is logged in
            if (loggedInUser) {
                loadUserData(loggedInUser);
            }
        };
        websocket.onmessage = (event) => {
            try {
                const data = JSON.parse(event.data);
                // Update state based on received data
                if (data.meals) setMeals(data.meals);
                if (data.chores) setChores(data.chores);
                if (data.ingredientPrices) setIngredientPrices(data.ingredientPrices);
                if (data.householdItems) setHouseholdItems(data.householdItems);
                if (data.checklist) setChecklist(data.checklist);
                if (data.checklistItems) setChecklistItems(data.checklistItems);
                if (data.currency) setCurrency(data.currency);
                if (data.backgroundImage) setBackgroundImage(data.backgroundImage);
                if (data.ingredientQuantities) {
                    localStorage.setItem('ingredientQuantities', JSON.stringify(data.ingredientQuantities));
                }
                if (data.removedIngredients) {
                    localStorage.setItem('removedIngredients', JSON.stringify(data.removedIngredients));
                }
            } catch (e) {
                console.error('Error parsing WebSocket message:', e);
            }
        };
        websocket.onerror = (error) => console.error('WebSocket error:', error);
        websocket.onclose = () => console.log('WebSocket closed');
        setWs(websocket);

        return () => {
            websocket.close();
        };
    }, []);

    // Function to broadcast state to other devices
    const broadcastState = (state) => {
        if (ws && ws.readyState === WebSocket.OPEN && loggedInUser) {
            ws.send(JSON.stringify({ username: loggedInUser, ...state }));
        }
    };

    const loadUserData = (user) => {
        try {
            const data = {
                meals: JSON.parse(localStorage.getItem(`${user}_meals`) || '{}'),
                chores: JSON.parse(localStorage.getItem(`${user}_chores`) || '[]'),
                ingredientPrices: JSON.parse(localStorage.getItem(`${user}_ingredientPrices`) || '{}'),
                householdItems: JSON.parse(localStorage.getItem(`${user}_householdItems`) || '[]'),
                checklist: JSON.parse(localStorage.getItem(`${user}_checklist`) || '{}'),
                checklistItems: JSON.parse(localStorage.getItem(`${user}_checklistItems`) || '[]'),
                currency: localStorage.getItem(`${user}_currency`) || 'USD',
                backgroundImage: localStorage.getItem(`${user}_backgroundImage`) || ''
            };
            setMeals(data.meals);
            setChores(data.chores);
            setIngredientPrices(data.ingredientPrices);
            setHouseholdItems(data.householdItems);
            setChecklist(data.checklist.week ? data.checklist : {
                week: Math.floor((new Date() - new Date(new Date().getFullYear(), 0, 1)) / (1000 * 60 * 60 * 24 * 7))
            });
            setChecklistItems(data.checklistItems);
            setCurrency(data.currency);
            setBackgroundImage(data.backgroundImage);
            // Broadcast initial state to sync devices
            broadcastState({
                meals: data.meals,
                chores: data.chores,
                ingredientPrices: data.ingredientPrices,
                householdItems: data.householdItems,
                checklist: data.checklist,
                checklistItems: data.checklistItems,
                currency: data.currency,
                backgroundImage: data.backgroundImage,
                ingredientQuantities: JSON.parse(localStorage.getItem('ingredientQuantities') || '{}'),
                removedIngredients: JSON.parse(localStorage.getItem('removedIngredients') || '[]')
            });
        } catch (e) {
            console.error('Error loading user data from localStorage:', e);
        }
    };

    const saveUserData = () => {
        if (!loggedInUser) return;
        try {
            localStorage.setItem(`${loggedInUser}_meals`, JSON.stringify(meals));
            localStorage.setItem(`${loggedInUser}_chores`, JSON.stringify(chores));
            localStorage.setItem(`${loggedInUser}_ingredientPrices`, JSON.stringify(ingredientPrices));
            localStorage.setItem(`${loggedInUser}_householdItems`, JSON.stringify(householdItems));
            localStorage.setItem(`${loggedInUser}_checklist`, JSON.stringify(checklist));
            localStorage.setItem(`${loggedInUser}_checklistItems`, JSON.stringify(checklistItems));
            localStorage.setItem(`${loggedInUser}_currency`, JSON.stringify(currency));
            localStorage.setItem(`${loggedInUser}_backgroundImage`, JSON.stringify(backgroundImage));
        } catch (e) {
            console.error('Error saving user data to localStorage:', e);
        }
    };

    useEffect(() => {
        if (loggedInUser) {
            loadUserData(loggedInUser);
        }
    }, [loggedInUser]);

    useEffect(() => {
        saveUserData();
    }, [meals, chores, ingredientPrices, householdItems, checklist, checklistItems, currency, backgroundImage, loggedInUser]);

    const handleLogout = () => {
        setLoggedInUser(null);
        setCurrentSection('calendar');
        setMeals({});
        setChores([]);
        setIngredientPrices({});
        setHouseholdItems([]);
        setChecklist({ week: Math.floor((new Date() - new Date(new Date().getFullYear(), 0, 1)) / (1000 * 60 * 60 * 24 * 7)) });
        setChecklistItems([]);
        setCurrency('USD');
        setBackgroundImage('');
        broadcastState({
            username: loggedInUser,
            meals: {},
            chores: [],
            ingredientPrices: {},
            householdItems: [],
            checklist: { week: Math.floor((new Date() - new Date(new Date().getFullYear(), 0, 1)) / (1000 * 60 * 60 * 24 * 7)) },
            checklistItems: [],
            currency: 'USD',
            backgroundImage: '',
            ingredientQuantities: {},
            removedIngredients: []
        });
    };

    if (!loggedInUser) {
        return <LoginRegister setLoggedInUser={setLoggedInUser} />;
    }

    return (
        <ErrorBoundary>
            <div 
                className="min-h-screen p-4"
                style={{
                    backgroundImage: backgroundImage ? `url(${backgroundImage})` : 'none',
                    backgroundSize: 'cover',
                    backgroundPosition: 'center',
                    backgroundAttachment: 'fixed'
                }}
            >
                <h1 className="text-3xl font-bold text-purple-600 text-center mb-6">Family Schedule</h1>
                <Navigation setCurrentSection={setCurrentSection} currentSection={currentSection} />
                <div className="max-w-7xl mx-auto mt-6">
                    {currentSection === 'calendar' && (
                        <CalendarOverview
                            meals={meals}
                            chores={chores}
                            setMeals={setMeals}
                            setChores={setChores}
                            checklist={checklist}
                            setChecklist={setChecklist}
                            checklistItems={checklistItems}
                            setChecklistItems={setChecklistItems}
                            householdItems={householdItems}
                            setHouseholdItems={setHouseholdItems}
                            currency={currency}
                            broadcastState={broadcastState}
                        />
                    )}
                    {currentSection === 'meals' && (
                        <MealsSection
                            setMeals={setMeals}
                            currency={currency}
                            meals={meals}
                            broadcastState={broadcastState}
                        />
                    )}
                    {currentSection === 'shopping' && (
                        <ShoppingList
                            meals={meals}
                            ingredientPrices={ingredientPrices}
                            setIngredientPrices={setIngredientPrices}
                            householdItems={householdItems}
                            setHouseholdItems={setHouseholdItems}
                            currency={currency}
                            broadcastState={broadcastState}
                        />
                    )}
                    {currentSection === 'chores' && (
                        <div className="space-y-4">
                            <ChoresForm setChores={setChores} broadcastState={broadcastState} />
                            <ChoresList chores={chores} setChores={setChores} broadcastState={broadcastState} />
                        </div>
                    )}
                    {currentSection === 'spending' && (
                        <MonthlySpending
                            meals={meals}
                            ingredientPrices={ingredientPrices}
                            householdItems={householdItems}
                            currency={currency}
                        />
                    )}
                    {currentSection === 'settings' && (
                        <Settings
                            currency={currency}
                            setCurrency={setCurrency}
                            backgroundImage={backgroundImage}
                            setBackgroundImage={setBackgroundImage}
                            handleLogout={handleLogout}
                            broadcastState={broadcastState}
                        />
                    )}
                </div>
            </div>
        </ErrorBoundary>
    );
};
        // Render the app
        ReactDOM.render(<App />, document.getElementById('root'));
    </script>
</body>
</html>
