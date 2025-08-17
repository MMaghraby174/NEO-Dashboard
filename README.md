import React, { useState, useEffect } from 'react';
import { BarChart, Bar, XAxis, YAxis, CartesianGrid, Tooltip, Legend, ResponsiveContainer, AreaChart, Area } from 'recharts';
import { Droplets, TrendingUp, CheckCircle, Brain, Target, HeartPulse, BedDouble, Activity, BarChart2, Flame, Zap, Leaf, Fish, Carrot, Egg, Shield, Sun, User, Upload, UtensilsCrossed, Dumbbell, Clock, Waves, Snowflake, Calendar, Watch, Check, X, Menu } from 'lucide-react';

// Firebase Imports for Firestore & Auth
import { initializeApp } from 'firebase/app';
import { getAuth, signInWithCustomToken, signInAnonymously } from 'firebase/auth';
import { getFirestore, onSnapshot, collection, query, orderBy, serverTimestamp } from 'firebase/firestore';

// --- Mock Data ---
const biomarkerData = [
  { name: 'Jan', glucose: 95, inflammation: 8.2, cholesterol: 210, testosterone: 550 },
  { name: 'Feb', glucose: 91, inflammation: 7.5, cholesterol: 202, testosterone: 560 },
  { name: 'Mar', glucose: 88, inflammation: 6.1, cholesterol: 195, testosterone: 580 },
  { name: 'Apr', glucose: 85, inflammation: 4.9, cholesterol: 188, testosterone: 610 },
  { name: 'May', glucose: 86, inflammation: 4.5, cholesterol: 185, testosterone: 620 },
  { name: 'Jun', glucose: 84, inflammation: 3.8, cholesterol: 180, testosterone: 640 },
];

const cgmData = [
  { time: '6am', glucose: 85 }, { time: '8am', glucose: 110 }, { time: '10am', glucose: 95 },
  { time: '12pm', glucose: 140 }, { time: '2pm', glucose: 115 }, { time: '4pm', glucose: 90 },
  { time: '6pm', glucose: 130 }, { time: '8pm', glucose: 105 }, { time: '10pm', glucose: 92 },
];

const foodScoreboardData = [
    { name: 'Avocado', icon: '🥑', score: 95, improves: ['Cholesterol', 'Glucose'] },
    { name: 'Salmon', icon: '🍣', score: 92, improves: ['Inflammation', 'Cognition'] },
    { name: 'Spinach', icon: '🥬', score: 88, improves: ['Iron', 'Energy'] },
    { name: 'Eggs', icon: '🍳', score: 85, improves: ['Testosterone', 'Vitamin D'] },
];

const mealPlanData = {
    breakfast: [
        { id: 'b1', name: 'Scrambled Eggs with Spinach & Avocado', calories: 350, protein: 25, carbs: 10, fats: 25, img: 'https://images.pexels.com/photos/566566/pexels-photo-566566.jpeg?auto=compress&cs=tinysrgb&w=1260&h=750&dpr=2', micronutrients: [{name: 'Vitamin D', value: '15mcg'}, {name: 'Iron', value: '4mg'}] },
        { id: 'b2', name: 'Greek Yogurt with Berries & Nuts', calories: 400, protein: 30, carbs: 35, fats: 15, img: 'https://images.pexels.com/photos/1092730/pexels-photo-1092730.jpeg?auto=compress&cs=tinysrgb&w=1260&h=750&dpr=2', micronutrients: [{name: 'Calcium', value: '200mg'}, {name: 'Magnesium', value: '50mg'}] },
        { id: 'b3', name: 'Smoothie with Protein Powder', calories: 300, protein: 35, carbs: 20, fats: 10, img: 'https://images.pexels.com/photos/1035661/pexels-photo-1035661.jpeg?auto=compress&cs=tinysrgb&w=1260&h=750&dpr=2', micronutrients: [{name: 'Potassium', value: '400mg'}, {name: 'Vitamin C', value: '30mg'}] },
    ],
    lunch: [
        { id: 'l1', name: 'Grilled Salmon with Quinoa & Asparagus', calories: 550, protein: 40, carbs: 40, fats: 25, img: 'https://images.pexels.com/photos/64208/pexels-photo-64208.jpeg?auto=compress&cs=tinysrgb&w=1260&h=750&dpr=2', micronutrients: [{name: 'Omega-3', value: '2.5g'}, {name: 'Vitamin B12', value: '5mcg'}] },
        { id: 'l2', name: 'Chicken Salad with Mixed Greens', calories: 450, protein: 35, carbs: 15, fats: 30, img: 'https://images.pexels.com/photos/1211887/pexels-photo-1211887.jpeg?auto=compress&cs=tinysrgb&w=1260&h=750&dpr=2', micronutrients: [{name: 'Vitamin K', value: '120mcg'}, {name: 'Selenium', value: '30mcg'}] },
        { id: 'l3', name: 'Tuna and White Bean Salad', calories: 400, protein: 30, carbs: 20, fats: 20, img: 'https://images.pexels.com/photos/2097090/pexels-photo-2097090.jpeg?auto=compress&cs=tinysrgb&w=1260&h=750&dpr=2', micronutrients: [{name: 'Folate', value: '200mcg'}, {name: 'Zinc', value: '5mg'}] },
    ],
    dinner: [
        { id: 'd1', name: 'Lean Steak with Sweet Potato & Broccoli', calories: 600, protein: 50, carbs: 45, fats: 25, img: 'https://images.pexels.com/photos/769289/pexels-photo-769289.jpeg?auto=compress&cs=tinysrgb&w=1260&h=750&dpr=2', micronutrients: [{name: 'Zinc', value: '10mg'}, {name: 'Iron', value: '6mg'}] },
        { id: 'd2', name: 'Lentil Soup with Whole Grain Bread', calories: 500, protein: 25, carbs: 70, fats: 10, img: 'https://images.pexels.com/photos/539451/pexels-photo-539451.jpeg?auto=compress&cs=tinysrgb&w=1260&h=750&dpr=2', micronutrients: [{name: 'Fiber', value: '15g'}, {name: 'Folate', value: '400mcg'}] },
        { id: 'd3', name: 'Chicken and Vegetable Stir-fry', calories: 500, protein: 45, carbs: 30, fats: 20, img: 'https://images.pexels.com/photos/70497/pexels-photo-70497.jpeg?auto=compress&cs=tinysrgb&w=1260&h=750&dpr=2', micronutrients: [{name: 'Vitamin A', value: '900mcg'}, {name: 'Magnesium', value: '80mg'}] },
    ]
};

const availableDates = ['Mon, Aug 19', 'Tue, Aug 20', 'Wed, Aug 21', 'Thu, Aug 22'];
const timeSlots = ['9:00 AM', '10:30 AM', '1:00 PM', '2:30 PM', '4:00 PM'];
const wearableOptions = [
    { name: 'Oura Ring', icon: '💍', status: 'Connected' },
    { name: 'Apple Watch', icon: '⌚️', status: 'Not Connected' },
    { name: 'Fitbit', icon: '👣', status: 'Not Connected' },
    { name: 'Whoop', icon: '💪', status: 'Not Connected' }
];

const organHealthData = [
  { name: 'Brain', score: 92, status: 'Optimal', position: { top: '10%', left: '50%' } },
  { name: 'Cardiovascular', score: 88, status: 'Optimal', position: { top: '35%', left: '42%' } },
  { name: 'Gut', score: 75, status: 'Needs Improvement', position: { top: '55%', left: '55%' } },
  { name: 'Metabolism', score: 82, status: 'Optimal', position: { top: '65%', left: '45%' } },
  { name: 'Skin', score: 95, status: 'Optimal', position: { top: '30%', left: '70%' } },
  { name: 'Hormonal', score: 70, status: 'Needs Improvement', position: { top: '45%', left: '65%' } },
];

const generateMockMeals = () => {
    const meals = [];
    const today = new Date();

    for (let i = 0; i < 15; i++) {
        const date = new Date(today);
        date.setDate(today.getDate() - i);
        const dayOfWeek = date.getDay();

        // Add meals on certain days to demonstrate the highlight feature
        if (dayOfWeek === 1 || dayOfWeek === 3 || dayOfWeek === 5 || dayOfWeek === 0) { // Monday, Wednesday, Friday, Sunday
            meals.push({
                id: `meal-${date.toISOString()}-breakfast`,
                name: 'Scrambled Eggs and Avocado',
                createdAt: { toDate: () => new Date(date.setHours(9, 0, 0)) },
                img: 'https://images.pexels.com/photos/566566/pexels-photo-566566.jpeg?auto=compress&cs=tinysrgb&w=1260&h=750&dpr=2',
                items: [{ food_item: 'Eggs', nutrients: { protein_g: 12, carbs_g: 1, fat_g: 10 } }]
            });
            meals.push({
                id: `meal-${date.toISOString()}-lunch`,
                name: 'Grilled Salmon with Quinoa',
                createdAt: { toDate: () => new Date(date.setHours(13, 30, 0)) },
                img: 'https://images.pexels.com/photos/64208/pexels-photo-64208.jpeg?auto=compress&cs=tinysrgb&w=1260&h=750&dpr=2',
                items: [{ food_item: 'Salmon', nutrients: { protein_g: 25, carbs_g: 5, fat_g: 15 } }]
            });
            if (dayOfWeek === 1 || dayOfWeek === 5) { // Add a third meal on Mondays and Fridays
                 meals.push({
                    id: `meal-${date.toISOString()}-dinner`,
                    name: 'Chicken and Vegetable Stir-fry',
                    createdAt: { toDate: () => new Date(date.setHours(19, 0, 0)) },
                    img: 'https://images.pexels.com/photos/70497/pexels-photo-70497.jpeg?auto=compress&cs=tinysrgb&w=1260&h=750&dpr=2',
                    items: [{ food_item: 'Chicken', nutrients: { protein_g: 30, carbs_g: 10, fat_g: 5 } }]
                });
            }
        }
    }
    return meals;
};

const mockLoggedMeals = generateMockMeals();


// --- UI Components ---

const DashboardCard = ({ title, icon, children, className }) => (
  <div className={`bg-[#1f2937] border border-[#374151] rounded-xl p-6 shadow-lg ${className}`}>
    <div className="flex items-center mb-4">
      {icon}
      <h3 className="text-lg font-semibold text-white ml-3">{title}</h3>
    </div>
    {children}
  </div>
);

const HealthScore = () => (
  <div className="bg-gradient-to-br from-green-800 to-green-900 border border-green-700 rounded-xl p-6 text-center flex flex-col justify-between">
    <div>
      <h3 className="text-lg font-semibold text-white mb-2">Overall Health Score</h3>
      <p className="text-6xl font-bold text-white tracking-tighter">88<span className="text-3xl text-green-300">/100</span></p>
    </div>
    <div className="flex items-center justify-center mt-4 text-green-200">
      <TrendingUp size={20} className="mr-2" />
      <p>Up 12 points since Jan 2025</p>
    </div>
  </div>
);

const WearableMetricCard = ({ title, value, unit, Icon, status, statusColor }) => (
    <div className="bg-[#2a3b4d] p-4 rounded-lg">
        <div className="flex items-center text-gray-300 mb-2">
            <Icon size={16} className="mr-2" />
            <span className="text-sm font-medium">{title}</span>
        </div>
        <p className="text-2xl font-bold text-white">{value} <span className="text-base font-normal text-gray-400">{unit}</span></p>
        <p className="text-sm font-semibold" style={{ color: statusColor }}>{status}</p>
    </div>
);

const BiomarkerRangeBar = ({ value, optimalRange }) => {
    const fullRange = optimalRange.max * 1.5;
    const position = Math.max(0, Math.min(100, (value / fullRange) * 100));
    const optimalStart = (optimalRange.min / fullRange) * 100;
    const optimalWidth = ((optimalRange.max - optimalRange.min) / fullRange) * 100;

    return (
        <div>
            <div className="relative h-2 w-full bg-gray-700 rounded-full">
                <div className="absolute h-2 rounded-full bg-yellow-600" style={{ left: 0, width: `${optimalStart}%` }}></div>
                <div className="absolute h-2 rounded-full bg-green-600" style={{ left: `${optimalStart}%`, width: `${optimalWidth}%` }}></div>
                <div className="absolute h-2 rounded-full bg-red-600" style={{ left: `${optimalStart + optimalWidth}%`, right: 0 }}></div>
                <div className="absolute top-1/2 -translate-y-1/2 w-3 h-3 bg-white rounded-full border-2 border-gray-900" style={{ left: `calc(${position}% - 6px)` }}></div>
            </div>
            <div className="text-xs text-gray-400 text-center mt-1">
                Optimal: {optimalRange.min} - {optimalRange.max}
            </div>
        </div>
    );
};

const BiomarkerRow = ({ name, value, unit, status, optimalRange }) => {
    const statusStyles = {
        Optimal: 'bg-green-800 text-green-300',
        'Needs Improvement': 'bg-yellow-800 text-yellow-300',
        'At Risk': 'bg-red-800 text-red-300',
    };
    return (
        <div className="grid grid-cols-12 items-center py-3 border-b border-gray-700 last:border-b-0">
            <div className="col-span-3 text-sm text-gray-300">{name}</div>
            <div className="col-span-2 text-lg font-bold text-white">{value} <span className="text-sm text-gray-400">{unit}</span></div>
            <div className="col-span-5">
                <BiomarkerRangeBar value={value} optimalRange={optimalRange} />
            </div>
            <div className="col-span-2 text-right">
                <span className={`px-2 py-1 text-xs font-semibold rounded-full ${statusStyles[status]}`}>{status}</span>
            </div>
        </div>
    );
};

const MealCard = ({ meal, isSelected, onSelect }) => (
    <div className={`relative bg-[#2a3b4d] rounded-lg overflow-hidden border-2 ${isSelected ? 'border-green-500' : 'border-transparent'} cursor-pointer transition-all group`} onClick={() => onSelect(meal.id)}>
        <img src={meal.img} alt={meal.name} className="w-full h-32 object-cover" onError={(e) => {e.target.onerror = null; e.target.src='https://placehold.co/300x200/1f2937/9ca3af?text=Image+Error'}} />
        <div className="p-4">
            <h4 className="font-semibold text-white">{meal.name}</h4>
            <div className="text-xs text-gray-400 mt-2 flex justify-between">
                <span>{meal.calories} kcal</span>
                <span>P: {meal.protein}g</span>
                <span>C: {meal.carbs}g</span>
                <span>F: {meal.fats}g</span>
            </div>
            <div className="border-t border-gray-600 mt-3 pt-3">
                 <h5 className="text-xs font-bold text-gray-300 mb-1">Key Micronutrients</h5>
                 <div className="text-xs text-gray-400 flex justify-between">
                    {meal.micronutrients.map(micro => <span key={micro.name}>{micro.name}: <strong>{micro.value}</strong></span>)}
                 </div>
            </div>
        </div>
        {isSelected && <div className="absolute top-2 right-2 bg-green-500 rounded-full p-1 shadow-lg"><CheckCircle size={16} className="text-white"/></div>}
    </div>
);

const LifestyleCard = ({ title, icon, description, imageUrl, isGenerating, isComplete }) => (
    <div className="bg-[#2a3b4d] rounded-lg p-6 flex flex-col items-center text-center">
        {isGenerating && (
            <div className="w-full h-40 bg-gray-800 animate-pulse rounded-md flex items-center justify-center">
                <p className="text-gray-400">Generating image...</p>
            </div>
        )}
        {!isGenerating && imageUrl && (
             <img src={imageUrl} alt={title} className="w-full h-40 object-cover rounded-md mb-4" onError={(e) => {e.target.onerror = null; e.target.src='https://placehold.co/400x200/1f2937/9ca3af?text=Image+Error'}} />
        )}
        <div className="mt-2">
            <div className="flex items-center justify-center mb-2">
                {icon}
                <h4 className="text-lg font-semibold text-white ml-2">{title}</h4>
            </div>
            <p className="text-sm text-gray-400">{description}</p>
        </div>
    </div>
);

const LabBooking = ({ onBack }) => {
    const [selectedDate, setSelectedDate] = useState(null);
    const [selectedTime, setSelectedTime] = useState(null);
    const [isConfirmed, setIsConfirmed] = useState(false);

    const handleConfirm = () => {
        setIsConfirmed(true);
    };

    return (
        <DashboardCard title="Book a Lab Appointment" icon={<Calendar className="text-green-500" />}>
            {!isConfirmed ? (
                <div>
                    <h3 className="text-lg font-semibold text-white mb-4">1. Select a Date</h3>
                    <div className="grid grid-cols-2 md:grid-cols-4 gap-4 mb-8">
                        {availableDates.map(date => (
                            <button
                                key={date}
                                onClick={() => setSelectedDate(date)}
                                className={`p-4 rounded-lg border-2 ${selectedDate === date ? 'bg-green-700 border-green-500' : 'bg-[#2a3b4d] border-transparent'} hover:bg-green-700 transition-colors`}
                            >
                                <span className="block font-semibold text-sm">{date.split(',')[0]}</span>
                                <span className="block text-xs text-gray-400">{date.split(',')[1].trim()}</span>
                            </button>
                        ))}
                    </div>

                    <h3 className="text-lg font-semibold text-white mb-4">2. Select a Time Slot</h3>
                    <div className="grid grid-cols-2 md:grid-cols-3 gap-4 mb-8">
                        {timeSlots.map(time => (
                            <button
                                key={time}
                                onClick={() => setSelectedTime(time)}
                                className={`p-4 rounded-lg border-2 ${selectedTime === time ? 'bg-green-700 border-green-500' : 'bg-[#2a3b4d] border-transparent'} hover:bg-green-700 transition-colors`}
                                disabled={!selectedDate}
                            >
                                <span className="font-semibold">{time}</span>
                            </button>
                        ))}
                    </div>
                    <div className="flex justify-end space-x-4">
                        <button onClick={onBack} className="bg-gray-700 hover:bg-gray-600 text-white font-bold py-2 px-4 rounded-lg transition-colors">Back to Dashboard</button>
                        <button onClick={handleConfirm} disabled={!selectedDate || !selectedTime} className={`py-2 px-4 rounded-lg font-bold transition-colors ${selectedDate && selectedTime ? 'bg-green-700 hover:bg-green-600 text-white' : 'bg-gray-800 text-gray-500 cursor-not-allowed'}`}>
                            Confirm Appointment
                        </button>
                    </div>
                </div>
            ) : (
                <div className="text-center p-8">
                    <CheckCircle size={64} className="text-green-500 mx-auto mb-4" />
                    <h3 className="text-2xl font-bold text-white mb-2">Appointment Confirmed!</h3>
                    <p className="text-lg text-gray-300">Your appointment is booked for:</p>
                    <p className="text-xl font-bold text-white mt-2">{selectedDate} at {selectedTime}</p>
                    <button onClick={onBack} className="mt-8 bg-green-700 hover:bg-green-600 text-white font-bold py-2 px-4 rounded-lg transition-colors">Return to Dashboard</button>
                </div>
            )}
        </DashboardCard>
    );
};

const WearableIntegrationModal = ({ onClose }) => {
    return (
        <div className="fixed inset-0 bg-black bg-opacity-70 flex items-center justify-center p-4 z-50">
            <div className="bg-[#1f2937] border border-[#374151] rounded-xl p-8 max-w-lg w-full relative">
                <button onClick={onClose} className="absolute top-4 right-4 text-gray-400 hover:text-white transition-colors">
                    <X size={24} />
                </button>
                <h2 className="text-2xl font-bold text-white mb-6 text-center">Integrate Your Wearables</h2>
                <p className="text-gray-400 text-center mb-8">Connect your wearable devices to automatically sync your health data.</p>
                <div className="space-y-4">
                    {wearableOptions.map(device => (
                        <div key={device.name} className="bg-[#2a3b4d] p-4 rounded-lg flex items-center justify-between">
                            <div className="flex items-center">
                                <span className="text-2xl mr-4">{device.icon}</span>
                                <span className="font-semibold text-white">{device.name}</span>
                            </div>
                            <button
                                className={`px-4 py-2 rounded-full text-xs font-bold transition-colors ${device.status === 'Connected' ? 'bg-green-700 text-white' : 'bg-gray-700 hover:bg-green-600 text-gray-300'}`}
                                disabled={device.status === 'Connected'}
                            >
                                {device.status === 'Connected' ? 'Connected' : 'Connect'}
                            </button>
                        </div>
                    ))}
                </div>
            </div>
        </div>
    );
};

const BodySystemHealth = () => {
    const bodySvg = (
        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 160 300" className="w-full h-auto">
            <style>
                {`
                    .body-fill {
                        fill: #22c55e;
                        filter: drop-shadow(0 0 10px #22c55e);
                        opacity: 0.15;
                    }
                    @keyframes pulse {
                        0%, 100% { opacity: 0.15; }
                        50% { opacity: 0.3; }
                    }
                    .pulsing-body {
                        animation: pulse 4s infinite ease-in-out;
                    }
                `}
            </style>
            <path className="body-fill pulsing-body" d="M80 0 A30 30 0 1 1 80 60 A30 30 0 1 1 80 0 Z M65 60 C55 60 45 70 45 80 L45 150 C45 160 55 170 65 170 L95 170 C105 170 115 160 115 150 L115 80 C115 70 105 60 95 60 L65 60 Z M45 150 L40 180 L35 210 L35 250 L45 250 L45 210 L50 180 Z M115 150 L120 180 L125 210 L125 250 L115 250 L115 210 L110 180 Z M45 250 L40 280 L40 300 L55 300 L55 280 L60 250 Z M115 250 L120 280 L120 300 L105 300 L105 280 L100 250 Z M65 170 L55 200 L55 250 L65 250 L65 200 Z M95 170 L105 200 L105 250 L95 250 L95 200 Z" />
        </svg>
    );

    const statusColors = {
        'Optimal': 'border-green-500 bg-green-900 shadow-[0_0_8px_rgb(34,197,94)]',
        'Needs Improvement': 'border-yellow-500 bg-yellow-900 shadow-[0_0_8px_rgb(234,179,8)]',
        'At Risk': 'border-red-500 bg-red-900 shadow-[0_0_8px_rgb(239,68,68)]',
    };

    return (
        <div className="relative w-full max-w-sm mx-auto">
            {bodySvg}
            
            {organHealthData.map((organ, index) => (
                <div
                    key={index}
                    className={`absolute rounded-xl p-3 shadow-md border-2 text-center transition-all hover:scale-105 transform ${statusColors[organ.status]}`}
                    style={{
                        top: organ.position.top,
                        left: organ.position.left,
                        transform: 'translate(-50%, -50%)',
                        minWidth: '120px'
                    }}
                >
                    <p className="font-semibold text-white text-sm">{organ.name}</p>
                    <p className="text-xs text-gray-300 mt-1">Score: <span className="font-bold text-lg">{organ.score}</span></p>
                </div>
            ))}
        </div>
    );
};

const CollapsibleMeal = ({ meal, expandedMeal, setExpandedMeal }) => {
    const isExpanded = expandedMeal === meal.id;

    const calculateTotalNutrients = (meal) => {
        if (!meal.items) return { protein_g: 0, carbs_g: 0, fat_g: 0, vitamin_c_mg: 0, iron_mg: 0, calcium_mg: 0 };
        return meal.items.reduce((totals, item) => {
            if (item.nutrients) {
                totals.protein_g += item.nutrients.protein_g || 0;
                totals.carbs_g += item.nutrients.carbs_g || 0;
                totals.fat_g += item.nutrients.fat_g || 0;
                totals.vitamin_c_mg += item.nutrients.vitamin_c_mg || 0;
                totals.iron_mg += item.nutrients.iron_mg || 0;
                totals.calcium_mg += item.nutrients.calcium_mg || 0;
            }
            return totals;
        }, { protein_g: 0, carbs_g: 0, fat_g: 0, vitamin_c_mg: 0, iron_mg: 0, calcium_mg: 0 });
    };

    const totals = calculateTotalNutrients(meal);
    const mealDate = meal.createdAt?.toDate ? meal.createdAt.toDate() : null;
    const formattedDate = mealDate ? mealDate.toLocaleDateString('en-US', { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' }) : 'Loading...';

    return (
      <div className="bg-[#2a3b4d] rounded-xl shadow-lg p-6 mb-4">
        <button
          onClick={() => setExpandedMeal(isExpanded ? null : meal.id)}
          className="flex justify-between items-center w-full focus:outline-none"
        >
          <div className="flex flex-col items-start">
            <h3 className="font-semibold text-lg text-white">{meal.name}</h3>
            <p className="text-sm text-gray-400">
              {formattedDate}
            </p>
          </div>
          <Menu className={`h-6 w-6 text-gray-400 transition-transform duration-300 ${isExpanded ? 'transform rotate-90' : ''}`} />
        </button>
        {isExpanded && (
          <div className="mt-4 border-t border-gray-600 pt-4">
            {meal.img && (
              <img 
                src={meal.img} 
                alt={meal.name} 
                className="w-full h-48 object-cover rounded-md mb-4" 
                onError={(e) => {e.target.onerror = null; e.target.src='https://placehold.co/600x400/1f2937/9ca3af?text=Image+Not+Available'}} 
              />
            )}
            <h4 className="font-semibold text-white mb-2">Total Nutrients</h4>
            <div className="grid grid-cols-2 gap-2 text-gray-400 text-sm">
              <p>Protein: <span className="font-bold text-green-300">{totals.protein_g.toFixed(1)}g</span></p>
              <p>Carbs: <span className="font-bold text-green-300">{totals.carbs_g.toFixed(1)}g</span></p>
              <p>Fat: <span className="font-bold text-green-300">{totals.fat_g.toFixed(1)}g</span></p>
              <p>Vitamin C: <span className="font-bold text-green-300">{totals.vitamin_c_mg.toFixed(1)}mg</span></p>
              <p>Iron: <span className="font-bold text-green-300">{totals.iron_mg.toFixed(1)}mg</span></p>
              <p>Calcium: <span className="font-bold text-green-300">{totals.calcium_mg.toFixed(1)}mg</span></p>
            </div>
            <h4 className="font-semibold text-white mt-4 mb-2">Meal Items</h4>
            <ul className="list-disc list-inside text-gray-400 text-sm pl-4">
              {meal.items.map((item, index) => (
                <li key={index} className="mb-1">
                  <span className="font-medium text-white">{item.food_item}</span>
                  <p className="ml-5 text-xs text-gray-500">
                    P: {item.nutrients.protein_g?.toFixed(1) || 0}g, C: {item.nutrients.carbs_g?.toFixed(1) || 0}g, F: {item.nutrients.fat_g?.toFixed(1) || 0}g
                  </p>
                </li>
              ))}
            </ul>
          </div>
        )}
      </div>
    );
};

// Component for the calendar date picker
const MealDatePicker = ({ loggedMeals, selectedDate, onSelectDate }) => {
    // Get the last 15 days
    const dates = Array.from({ length: 15 }, (_, i) => {
        const d = new Date();
        d.setDate(d.getDate() - i);
        return d;
    }).reverse();

    // Create a Set of dates that have logged meals for quick lookup
    const datesWithMeals = new Set(
        loggedMeals.map(meal => {
            if (meal.createdAt && meal.createdAt.toDate) {
                return meal.createdAt.toDate().toISOString().slice(0, 10);
            }
            return null;
        }).filter(Boolean)
    );

    return (
        <div className="grid grid-cols-5 md:grid-cols-8 lg:grid-cols-15 gap-2 mb-6 overflow-x-auto pb-2 -mx-2 px-2">
            {dates.map(date => {
                const dateString = date.toISOString().slice(0, 10);
                const day = date.getDate();
                const hasMeals = datesWithMeals.has(dateString);
                const isSelected = selectedDate === dateString;

                return (
                    <button
                        key={dateString}
                        onClick={() => onSelectDate(dateString)}
                        className={`
                            p-2 rounded-lg text-center
                            ${isSelected ? 'bg-green-600 text-white' : 'bg-[#2a3b4d] text-gray-300'}
                            ${hasMeals ? 'border-2 border-green-500' : 'border-2 border-transparent'}
                            hover:bg-green-700 hover:text-white transition-colors
                            flex-shrink-0 w-16 md:w-auto
                        `}
                    >
                        <span className="block font-semibold text-sm">{date.toLocaleDateString('en-US', { weekday: 'short' })}</span>
                        <span className="block text-2xl font-bold">{day}</span>
                    </button>
                );
            })}
        </div>
    );
};


// --- Main App Component ---
export default function App() {
  const [activeTab, setActiveTab] = useState('dashboard');
  const [selectedMeals, setSelectedMeals] = useState({ breakfast: null, lunch: null, dinner: null });
  const [showWearableModal, setShowWearableModal] = useState(false);
  const [allLoggedMeals, setAllLoggedMeals] = useState(mockLoggedMeals);
  const [expandedMeal, setExpandedMeal] = useState(null);
  
  const today = new Date();
  const todayDateString = today.toISOString().slice(0, 10);
  const [selectedDate, setSelectedDate] = useState(todayDateString);
  
  const [images, setImages] = useState({
      strength: null,
      fasting: null,
      sauna: null,
      coldPlunge: null
  });
  const [isGenerating, setIsGenerating] = useState(true);
  
  const [db, setDb] = useState(null);
  const [auth, setAuth] = useState(null);
  const [userId, setUserId] = useState(null);
  
  const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
  const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {};
  const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

  useEffect(() => {
    async function initFirebase() {
      try {
        const firebaseApp = initializeApp(firebaseConfig);
        const firestoreDb = getFirestore(firebaseApp);
        const firebaseAuth = getAuth(firebaseApp);

        setDb(firestoreDb);
        setAuth(firebaseAuth);

        if (initialAuthToken) {
          await signInWithCustomToken(firebaseAuth, initialAuthToken);
        } else {
          await signInAnonymously(firebaseAuth);
        }

        const user = firebaseAuth.currentUser;
        if (user) {
          setUserId(user.uid);
          console.log("Firebase signed in with user ID:", user.uid);
        } else {
          const tempUserId = `anon-${crypto.randomUUID()}`;
          setUserId(tempUserId);
          console.log("Signed in anonymously with temporary user ID:", tempUserId);
        }
      } catch (e) {
        console.error("Error during Firebase initialization or sign-in:", e);
      }
    }
    if (!db) {
      initFirebase();
    }
  }, [db, initialAuthToken, firebaseConfig]);

  useEffect(() => {
    if (db && userId) {
      const mealsCollectionPath = `artifacts/${appId}/users/${userId}/loggedMeals`;
      const q = query(collection(db, mealsCollectionPath), orderBy('createdAt', 'desc'));

      const unsubscribe = onSnapshot(q, (snapshot) => {
        const meals = snapshot.docs.map(doc => ({
          id: doc.id,
          ...doc.data()
        }));
        // Combine mock data and live data, avoiding duplicates
        const liveMealIds = new Set(meals.map(m => m.id));
        const combinedMeals = [...meals, ...mockLoggedMeals.filter(m => !liveMealIds.has(m.id))];
        setAllLoggedMeals(combinedMeals);
        console.log("Fetched " + meals.length + " meals from Firestore and combined with mock data.");
      }, (error) => {
        console.error("Error fetching documents: ", error);
      });

      return () => unsubscribe();
    }
  }, [db, userId, appId]);
  
  const filteredMeals = allLoggedMeals.filter(meal => {
      const mealDate = meal.createdAt?.toDate ? meal.createdAt.toDate().toISOString().slice(0, 10) : '';
      return mealDate === selectedDate;
  });

  const generateImage = async (prompt) => {
      for (let attempt = 0; attempt < 3; attempt++) {
          try {
              const payload = { instances: { prompt: prompt }, parameters: { "sampleCount": 1} };
              const apiKey = ""
              const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/imagen-3.0-generate-002:predict?key=${apiKey}`;

              const response = await fetch(apiUrl, {
                  method: 'POST',
                  headers: { 'Content-Type': 'application/json' },
                  body: JSON.stringify(payload)
              });

              if (!response.ok) {
                  throw new Error(`API call failed with status: ${response.status}`);
              }
              const result = await response.json();
              if (result.predictions && result.predictions.length > 0 && result.predictions[0].bytesBase64Encoded) {
                return `data:image/png;base64,${result.predictions[0].bytesBase64Encoded}`;
              } else {
                throw new Error("Invalid API response format");
              }
          } catch (error) {
              console.error(`Attempt ${attempt + 1}: Error generating image for prompt "${prompt}".`, error);
              if (attempt < 2) {
                  await new Promise(res => setTimeout(res, 2 ** attempt * 1000));
              } else {
                  return null;
              }
          }
      }
  };

  useEffect(() => {
      let isMounted = true;
      const loadImages = async () => {
          if (!isMounted) return;
          setIsGenerating(true);
          const newImages = {
              strength: await generateImage("A person doing a full-body strength training workout with free weights in a modern gym. High-quality, realistic photograph."),
              fasting: await generateImage("A person looking energetic and focused, sitting at a table with a glass of water, symbolizing an extended fasting period. Clean, minimalist aesthetic. High-quality, realistic photograph."),
              sauna: await generateImage("A person relaxing in a dimly lit, traditional wooden sauna, with a calm and serene expression. The scene is steamy and warm, with gentle light filtering in. High-quality, realistic photograph."),
              coldPlunge: await generateImage("A person fully submerged in an icy cold plunge tub, with steam rising from the water, conveying a sense of invigoration and resilience. The setting is clean and modern. High-quality, realistic photograph.")
          };

          if (isMounted) {
              setImages(newImages);
              setIsGenerating(false);
          }
      };

      loadImages();

      return () => {
          isMounted = false;
      };
  }, []);

  const handleMealSelect = (mealType, mealId) => {
      setSelectedMeals(prev => ({...prev, [mealType]: mealId}));
  };

  const TabButton = ({ tabId, children }) => (
      <button 
        className={`px-4 py-2 text-sm font-semibold rounded-md transition-colors ${activeTab === tabId ? 'bg-green-800 text-white' : 'text-gray-400 hover:bg-[#2a3b4d]'}`}
        onClick={() => setActiveTab(tabId)}
      >
        {children}
      </button>
  );

  return (
    <div className="min-h-screen bg-[#121212] text-gray-200 font-sans p-4 md:p-8">
      <div className="max-w-7xl mx-auto">
        <header className="flex flex-col md:flex-row justify-between items-start md:items-center mb-8">
          <div className="flex items-center">
            <img src="https://placehold.co/48x48/1f2937/9ca3af?text=A" alt="User Avatar" className="w-12 h-12 rounded-full mr-4" />
            <div>
                <h1 className="text-xl font-bold text-white">MM</h1>
                <p className="text-sm text-gray-400">Optimize Package Member</p>
                {userId && <p className="text-xs text-gray-500 mt-1">User ID: <span className="font-mono">{userId}</span></p>}
            </div>
            <div className="flex flex-wrap gap-2 mt-4 md:mt-0 md:ml-6">
                <button onClick={() => setActiveTab('booking')} className="bg-[#1f2937] border border-[#374151] text-white font-semibold px-4 py-2 rounded-lg hover:bg-[#2a3b4d] transition flex items-center text-sm">
                    <Calendar size={16} className="mr-2"/>
                    Book Lab Appointment
                </button>
                <button onClick={() => setShowWearableModal(true)} className="bg-[#1f2937] border border-[#374151] text-white font-semibold px-4 py-2 rounded-lg hover:bg-[#2a3b4d] transition flex items-center text-sm">
                    <Watch size={16} className="mr-2"/>
                    Integrate Wearables
                </button>
                 <button className="bg-[#1f2937] border border-[#374151] text-white font-semibold px-4 py-2 rounded-lg hover:bg-[#2a3b4d] transition flex items-center text-sm">
                    <Upload size={16} className="mr-2"/>
                    Upload Lab Results
                </button>
            </div>
          </div>
          <div className="flex items-center mt-4 md:mt-0 bg-[#1f2937] border border-[#374151] rounded-lg p-1">
            <TabButton tabId="dashboard">Progress Dashboard</TabButton>
            <TabButton tabId="meals">Meal Plan & Recommendations</TabButton>
            <TabButton tabId="logged-meals">Logged Meals</TabButton>
            <TabButton tabId="lifestyle">Lifestyle Protocol</TabButton>
          </div>
        </header>

        {activeTab === 'dashboard' && (
            <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
                <div className="lg:col-span-2 space-y-6">
                    <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
                    <HealthScore />
                    <div className="md:col-span-2 bg-[#1f2937] border border-[#374151] rounded-xl p-4 shadow-lg">
                        <h3 className="text-md font-semibold text-white mb-3 px-2">Today's Wearable Insights</h3>
                        <div className="grid grid-cols-2 sm:grid-cols-2 gap-3">
                            <WearableMetricCard title="Sleep Score" value="92" unit="/ 100" Icon={BedDouble} status="Optimal" statusColor="#22c55e" />
                            <WearableMetricCard title="HRV" value="78" unit="ms" Icon={HeartPulse} status="Optimal" statusColor="#22c55e" />
                            <WearableMetricCard title="Resting HR" value="52" unit="bpm" Icon={Activity} status="Excellent" statusColor="#22c55e" />
                            <WearableMetricCard title="Activity" value="4.5" unit="/ 6" Icon={BarChart2} status="On Track" statusColor="#a3e635" />
                        </div>
                    </div>
                    </div>
                    
                    <DashboardCard title="Biomarker Health Overview" icon={<CheckCircle className="text-green-500" />}>
                        <div>
                            <div className="mb-4">
                                <h4 className="font-semibold text-white flex items-center mb-2"><Flame size={18} className="mr-2 text-red-500"/>Inflammation</h4>
                                <BiomarkerRow name="hs-CRP" value={0.8} unit="mg/L" status="Optimal" optimalRange={{ min: 0, max: 1.0 }} />
                                <BiomarkerRow name="White Blood Cells" value={6.2} unit="x10E9/L" status="Optimal" optimalRange={{ min: 4.0, max: 11.0 }} />
                            </div>
                            <div className="mb-4">
                                <h4 className="font-semibold text-white flex items-center mb-2"><Droplets size={18} className="mr-2 text-blue-500"/>Metabolism</h4>
                                <BiomarkerRow name="Glucose" value={84} unit="mg/dL" status="Optimal" optimalRange={{ min: 70, max: 100 }} />
                                <BiomarkerRow name="HbA1c" value={5.1} unit="%" status="Optimal" optimalRange={{ min: 4.0, max: 5.6 }} />
                            </div>
                             <div className="mb-4">
                                <h4 className="font-semibold text-white flex items-center mb-2"><Zap size={18} className="mr-2 text-yellow-500"/>Hormonal Health</h4>
                                <BiomarkerRow name="Testosterone" value={640} unit="ng/dL" status="Optimal" optimalRange={{ min: 300, max: 1000 }} />
                                <BiomarkerRow name="Cortisol (AM)" value={15} unit="ug/dL" status="Optimal" optimalRange={{ min: 6, max: 18 }} />
                            </div>
                             <div className="mb-4">
                                <h4 className="font-semibold text-white flex items-center mb-2"><Shield size={18} className="mr-2 text-indigo-400"/>Liver Function</h4>
                                <BiomarkerRow name="ALT" value={25} unit="U/L" status="Optimal" optimalRange={{ min: 7, max: 55 }} />
                            </div>
                            <div>
                                <h4 className="font-semibold text-white flex items-center mb-2"><Sun size={18} className="mr-2 text-amber-400"/>Key Nutrients</h4>
                                <BiomarkerRow name="Vitamin D" value={45} unit="ng/mL" status="Needs Improvement" optimalRange={{ min: 50, max: 80 }} />
                                <BiomarkerRow name="Ferritin (Iron)" value={150} unit="ng/mL" status="Optimal" optimalRange={{ min: 30, max: 300 }} />
                            </div>
                        </div>
                    </DashboardCard>
                </div>

                <div className="space-y-6">
                    <DashboardCard title="Body System Health" icon={<Target className="text-green-500" />}>
                        <BodySystemHealth />
                    </DashboardCard>
                    <DashboardCard title="Continuous Glucose (CGM) Trend - Last 24h" icon={<TrendingUp className="text-green-500" />}>
                    <ResponsiveContainer width="100%" height={200}>
                        <AreaChart data={cgmData} margin={{ top: 10, right: 20, left: -20, bottom: 0 }}>
                        <defs><linearGradient id="colorGlucose" x1="0" y1="0" x2="0" y2="1"><stop offset="5%" stopColor="#166534" stopOpacity={0.8}/><stop offset="95%" stopColor="#166534" stopOpacity={0}/></linearGradient></defs>
                        <CartesianGrid strokeDasharray="3 3" stroke="#374151" />
                        <XAxis dataKey="time" stroke="#9ca3af" fontSize={12} />
                        <YAxis domain={[60, 160]} stroke="#9ca3af" fontSize={12} unit=" mg/dL" />
                        <Tooltip contentStyle={{ backgroundColor: '#1f2937', border: '1px solid #374151' }} />
                        <Area type="monotone" dataKey="glucose" stroke="#22c55e" fillOpacity={1} fill="url(#colorGlucose)" />
                        </AreaChart>
                    </ResponsiveContainer>
                    </DashboardCard>
                     <DashboardCard title="Personalized Food Scoreboard" icon={<Leaf className="text-green-500" />}>
                        <p className="text-sm text-gray-400 mb-4">Top food recommendations to optimize your biomarkers.</p>
                        <div className="space-y-3">
                            {foodScoreboardData.map(food => (
                                <div key={food.name} className="bg-[#2a3b4d] p-4 rounded-lg flex items-center">
                                    <div className="text-3xl mr-4">{food.icon}</div>
                                    <div>
                                        <p className="font-semibold text-white">{food.name} <span className="text-base text-gray-400 ml-2">{food.score}/100</span></p>
                                        <p className="text-xs text-gray-400">Improves: {food.improves.join(', ')}</p>
                                    </div>
                                </div>
                            ))}
                        </div>
                    </DashboardCard>
                </div>
            </div>
        )}

        {activeTab === 'meals' && (
            <div>
                <DashboardCard title="Your Meal Recommendations for the Week" icon={<UtensilsCrossed className="text-green-500" />}>
                    <div className="space-y-8">
                        <div>
                            <h3 className="text-xl font-semibold text-white mb-4">Breakfast Options</h3>
                            <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
                                {mealPlanData.breakfast.map(meal => <MealCard key={meal.id} meal={meal} isSelected={selectedMeals.breakfast === meal.id} onSelect={() => handleMealSelect('breakfast', meal.id)} />)}
                            </div>
                        </div>
                         <div>
                            <h3 className="text-xl font-semibold text-white mb-4">Lunch Options</h3>
                            <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
                                {mealPlanData.lunch.map(meal => <MealCard key={meal.id} meal={meal} isSelected={selectedMeals.lunch === meal.id} onSelect={() => handleMealSelect('lunch', meal.id)} />)}
                            </div>
                        </div>
                         <div>
                            <h3 className="text-xl font-semibold text-white mb-4">Dinner Options</h3>
                            <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
                                {mealPlanData.dinner.map(meal => <MealCard key={meal.id} meal={meal} isSelected={selectedMeals.dinner === meal.id} onSelect={() => handleMealSelect('dinner', meal.id)} />)}
                            </div>
                        </div>
                    </div>
                     <div className="mt-8 text-right">
                        <button className="bg-green-700 hover:bg-green-600 text-white font-bold py-3 px-6 rounded-lg transition-colors">Confirm Selections for the Week</button>
                    </div>
                </DashboardCard>
            </div>
        )}

        {activeTab === 'logged-meals' && (
            <DashboardCard title="Logged Meals" icon={<UtensilsCrossed className="text-green-500" />}>
                <MealDatePicker
                    loggedMeals={allLoggedMeals}
                    selectedDate={selectedDate}
                    onSelectDate={setSelectedDate}
                />
                {filteredMeals.length > 0 ? (
                    filteredMeals.map(meal => (
                        <CollapsibleMeal key={meal.id} meal={meal} expandedMeal={expandedMeal} setExpandedMeal={setExpandedMeal} />
                    ))
                ) : (
                    <div className="text-center text-gray-400 mt-8">
                        <p>No meals have been logged for this date.</p>
                    </div>
                )}
            </DashboardCard>
        )}

        {activeTab === 'lifestyle' && (
             <DashboardCard title="Your Lifestyle Protocol" icon={<Activity className="text-green-500" />}>
                 <p className="text-gray-400 mb-6">These recommendations are tailored to your lab results to help optimize your health markers.</p>
                 <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
                     <LifestyleCard
                         title="Strength Training"
                         icon={<Dumbbell size={24} className="text-green-400" />}
                         description="Focus on compound lifts to increase testosterone and muscle mass. Aim for 3 sessions per week."
                         imageUrl={images.strength}
                         isGenerating={isGenerating}
                     />
                     <LifestyleCard
                         title="Intermittent Fasting"
                         icon={<Clock size={24} className="text-green-400" />}
                         description="Implement a 16:8 fasting window to improve glucose sensitivity and cellular repair."
                         imageUrl={images.fasting}
                         isGenerating={isGenerating}
                     />
                     <LifestyleCard
                         title="Sauna Therapy"
                         icon={<Waves size={24} className="text-green-400" />}
                         description="Use a sauna for 15-20 minutes after workouts to aid recovery and detoxification."
                         imageUrl={images.sauna}
                         isGenerating={isGenerating}
                     />
                     <LifestyleCard
                         title="Cold Plunge"
                         icon={<Snowflake size={24} className="text-green-400" />}
                         description="Incorporate a 3-5 minute cold plunge to reduce inflammation and boost circulation."
                         imageUrl={images.coldPlunge}
                         isGenerating={isGenerating}
                     />
                     <LifestyleCard
                         title="Wellness & Mindset"
                         icon={<Brain size={24} className="text-green-400" />}
                         description="Practice daily mindfulness meditation for 10 minutes to reduce cortisol levels and stress."
                         isGenerating={false}
                     />
                 </div>
             </DashboardCard>
        )}

        {activeTab === 'booking' && (
            <LabBooking onBack={() => setActiveTab('dashboard')} />
        )}

        {showWearableModal && <WearableIntegrationModal onClose={() => setShowWearableModal(false)} />}
      </div>
    </div>
  );
}
