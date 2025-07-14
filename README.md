<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>The Foresters Arms - Weekly Rota</title>
    
    <!-- Tailwind CSS for styling -->
    <script src="https://cdn.tailwindcss.com"></script>
    
    <!-- Libraries for PDF generation -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    
    <!-- React Libraries -->
    <script src="https://unpkg.com/react@18/umd/react.development.js"></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
    
    <!-- Babel to transpile JSX in the browser -->
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    
    <style>
        body {
            font-family: 'Inter', sans-serif;
        }
        /* Hide the default time input icon */
        input[type="time"]::-webkit-calendar-picker-indicator {
            display: none;
        }
        input[type="time"] {
            -webkit-appearance: none;
            -moz-appearance: none;
            appearance: none;
            background: transparent;
            border: none;
            cursor: pointer;
        }
    </style>
</head>
<body class="bg-gray-100">

    <div id="root">
        <div id="loading" class="flex justify-center items-center h-screen text-xl text-gray-600">Loading Rota App...</div>
    </div>

    <script type="text/babel">
        const { useState, useEffect, useCallback, Fragment } = React;

        // --- HELPER FUNCTIONS & INITIAL DATA ---

        const getWeekStartDate = (date) => {
            const d = new Date(date);
            const day = d.getDay();
            const diff = d.getDate() - day + (day === 0 ? -6 : 1); // Adjust when day is Sunday
            d.setHours(0, 0, 0, 0);
            return new Date(d.setDate(diff));
        };

        const formatDate = (date) => new Intl.DateTimeFormat('en-GB', { day: '2-digit', month: 'short', year: 'numeric' }).format(date);
        
        const timeToMinutes = (timeStr) => {
            if (!timeStr) return 0;
            const [hours, minutes] = timeStr.split(':').map(Number);
            return hours * 60 + minutes;
        };

        const minutesToTime = (minutes) => {
            const h = Math.floor(minutes / 60) % 24;
            const m = minutes % 60;
            return `${String(h).padStart(2, '0')}:${String(m).padStart(2, '0')}`;
        };
        
        // Base64 encoded logo to ensure it always loads
        const logoSrc = "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAoAAAAFeCAYAAACLP3v7AAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsMAAA7DAcdvqGQAAP+lSURBVHhe7P13lB1Vdf/7O22nU6fT6WzT9E473Zk2bUoTEpCEBEIgvAsC4gIiKogKgiCCiIigKCIiioqCCoJAEARCCAmkQJq+TqfTTKfT6WzT2Wf/WHvttc+lU6fTqZ6e3/f5rLPW2mvvtdde+zzr+b7P+z4gIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIi-7-H+r/0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0--0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0-0--0-";

        const initialRotaData = {
            staff: [
                { name: "Jake", color: "#10B981" },
                { name: "Sarah", color: "#F59E0B" },
                { name: "Tom", color: "#3B82F6" },
                { name: "Emily", color: "#8B5CF6" }
            ],
            openingHours: {
                "Sunday": { open: "12:00", close: "21:00" },
                "Monday": { open: "16:00", close: "23:00" },
                "Tuesday": { open: "16:00", close: "23:00" },
                "Wednesday": { open: "16:00", close: "23:00" },
                "Thursday": { open: "16:00", close: "23:00" },
                "Friday": { open: "12:00", close: "01:00" },
                "Saturday": { open: "12:00", close: "01:00" }
            },
            shifts: [
                { id: 1, day: "Sunday", staff: "Jake", startTime: "12:00", endTime: "21:00", breakMinutes: 0, department: 'Bar' },
                { id: 2, day: "Monday", staff: "Jake", startTime: "16:00", endTime: "23:00", breakMinutes: 30, department: 'Bar' },
                { id: 3, day: "Tuesday", staff: "Jake", startTime: "16:00", endTime: "18:30", breakMinutes: 0, department: 'Bar' },
                { id: 4, day: "Tuesday", staff: "Sarah", startTime: "18:30", endTime: "23:00", breakMinutes: 0, department: 'Bar' },
                { id: 5, day: "Wednesday", staff: "Jake", startTime: "16:00", endTime: "19:30", breakMinutes: 0, department: 'Bar' },
                { id: 6, day: "Thursday", staff: "Jake", startTime: "16:00", endTime: "23:00", breakMinutes: 30, department: 'Bar' },
                { id: 7, day: "Friday", staff: "Jake", startTime: "12:00", endTime: "19:00", breakMinutes: 30, department: 'Bar' },
                { id: 8, day: "Friday", staff: "Sarah", startTime: "19:00", endTime: "01:00", breakMinutes: 30, department: 'Bar' },
                { id: 9, day: "Saturday", staff: "Tom", startTime: "12:00", endTime: "18:00", breakMinutes: 30, department: 'Bar' },
                { id: 10, day: "Saturday", staff: "Emily", startTime: "18:00", endTime: "01:00", breakMinutes: 30, department: 'Bar' },
                { id: 11, day: "Friday", staff: "Tom", startTime: "17:00", endTime: "22:00", breakMinutes: 30, department: 'Kitchen' },
                { id: 12, day: "Saturday", staff: "Sarah", startTime: "17:00", endTime: "22:00", breakMinutes: 30, department: 'Kitchen' },
            ]
        };

        const ShiftModal = ({ shift, day, staffList, onSave, onClose, onDelete, department }) => {
            const [formData, setFormData] = React.useState(shift || {
                day: day,
                staff: staffList[0]?.name || '',
                startTime: '12:00',
                endTime: '17:00',
                breakMinutes: 0,
                department: department
            });

            const handleChange = (e) => {
                const { name, value } = e.target;
                setFormData(prev => ({ ...prev, [name]: value }));
            };

            const handleSave = (e) => {
                e.preventDefault();
                onSave(formData);
            };

            return (
                <div className="fixed inset-0 bg-black bg-opacity-50 flex justify-center items-center z-50 p-4">
                    <div className="bg-white p-6 rounded-lg shadow-2xl w-full max-w-md">
                        <h2 className="text-2xl font-bold mb-1">{shift ? 'Edit Shift' : 'Add Shift'} for {day}</h2>
                        <p className="text-lg text-gray-600 mb-6">{formData.department}</p>
                        <form onSubmit={handleSave}>
                            <div className="mb-4">
                                <label className="block text-gray-700 mb-2">Staff Member</label>
                                <select name="staff" value={formData.staff} onChange={handleChange} className="w-full p-2 border rounded-md bg-white">
                                    {staffList.map(s => <option key={s.name} value={s.name}>{s.name}</option>)}
                                </select>
                            </div>
                            <div className="grid grid-cols-2 gap-4 mb-4">
                                <div>
                                    <label className="block text-gray-700 mb-2">Start Time</label>
                                    <input type="time" name="startTime" value={formData.startTime} onChange={handleChange} className="w-full p-2 border rounded-md" />
                                </div>
                                <div>
                                    <label className="block text-gray-700 mb-2">End Time</label>
                                    <input type="time" name="endTime" value={formData.endTime} onChange={handleChange} className="w-full p-2 border rounded-md" />
                                </div>
                            </div>
                            <div className="mb-6">
                                <label className="block text-gray-700 mb-2">Break (minutes)</label>
                                <input type="number" name="breakMinutes" value={formData.breakMinutes} onChange={handleChange} className="w-full p-2 border rounded-md" min="0" step="15" />
                            </div>
                            <div className="flex justify-between">
                                <div>
                                    {shift && (
                                        <button type="button" onClick={() => onDelete(shift.id)} className="bg-red-600 text-white font-bold py-2 px-4 rounded-lg hover:bg-red-700 transition-colors">
                                            Delete
                                        </button>
                                    )}
                                </div>
                                <div className="space-x-2">
                                    <button type="button" onClick={onClose} className="bg-gray-300 text-gray-800 font-bold py-2 px-4 rounded-lg hover:bg-gray-400 transition-colors">
                                        Cancel
                                    </button>
                                    <button type="submit" className="bg-green-600 text-white font-bold py-2 px-4 rounded-lg hover:bg-green-700 transition-colors">
                                        Save Shift
                                    </button>
                                </div>
                            </div>
                        </form>
                    </div>
                </div>
            );
        };

        const StaffModal = ({ staffList, onSave, onClose }) => {
            const [staff, setStaff] = React.useState(staffList);

            const handleNameChange = (index, newName) => {
                const updatedStaff = [...staff];
                updatedStaff[index].name = newName;
                setStaff(updatedStaff);
            };

            const handleColorChange = (index, newColor) => {
                const updatedStaff = [...staff];
                updatedStaff[index].color = newColor;
                setStaff(updatedStaff);
            };
            
            const addStaffMember = () => {
                setStaff([...staff, { name: 'New Staff', color: '#cccccc' }]);
            };
            
            const removeStaffMember = (index) => {
                setStaff(staff.filter((_, i) => i !== index));
            };

            return (
                <div className="fixed inset-0 bg-black bg-opacity-50 flex justify-center items-center z-50 p-4">
                    <div className="bg-white p-8 rounded-lg shadow-2xl w-full max-w-lg">
                        <h2 className="text-2xl font-bold mb-6">Manage Staff</h2>
                        <div className="space-y-4 max-h-96 overflow-y-auto pr-2">
                            {staff.map((s, index) => (
                                <div key={index} className="flex items-center space-x-4">
                                    <input type="color" value={s.color} onChange={(e) => handleColorChange(index, e.target.value)} className="w-10 h-10 rounded-md p-0 border-none cursor-pointer"/>
                                    <input type="text" value={s.name} onChange={(e) => handleNameChange(index, e.target.value)} className="flex-grow p-2 border rounded-md" />
                                     <button onClick={() => removeStaffMember(index)} className="text-red-500 hover:text-red-700 font-semibold">Remove</button>
                                </div>
                            ))}
                        </div>
                         <button onClick={addStaffMember} className="mt-4 bg-blue-500 text-white font-bold py-2 px-4 rounded-lg hover:bg-blue-600 transition-colors">
                            Add Staff
                        </button>
                        <div className="flex justify-end space-x-2 mt-6">
                            <button type="button" onClick={onClose} className="bg-gray-300 text-gray-800 font-bold py-2 px-4 rounded-lg hover:bg-gray-400 transition-colors">
                                Cancel
                            </button>
                            <button type="button" onClick={() => onSave(staff)} className="bg-green-600 text-white font-bold py-2 px-4 rounded-lg hover:bg-green-700 transition-colors">
                                Save Staff
                            </button>
                        </div>
                    </div>
                </div>
            );
        };

        const App = () => {
            const [currentDate, setCurrentDate] = useState(new Date());
            const [rotaData, setRotaData] = useState(() => {
                try {
                    const savedRota = localStorage.getItem('forestersRota');
                    return savedRota ? JSON.parse(savedRota) : { [formatDate(getWeekStartDate(new Date()))]: initialRotaData };
                } catch (error) {
                    console.error("Could not parse rota data from localStorage", error);
                    return { [formatDate(getWeekStartDate(new Date()))]: initialRotaData };
                }
            });
            const [isShiftModalOpen, setIsShiftModalOpen] = useState(false);
            const [isStaffModalOpen, setIsStaffModalOpen] = useState(false);
            const [selectedShift, setSelectedShift] = useState(null);
            const [selectedDay, setSelectedDay] = useState('');
            const [selectedDepartment, setSelectedDepartment] = useState('Bar');

            const currentWeekKey = formatDate(getWeekStartDate(currentDate));

            useEffect(() => {
                try {
                    localStorage.setItem('forestersRota', JSON.stringify(rotaData));
                } catch (error) {
                    console.error("Could not save rota data to localStorage", error);
                }
            }, [rotaData]);

            useEffect(() => {
                if (!rotaData[currentWeekKey]) {
                    setRotaData(prev => {
                        const firstWeekKey = Object.keys(prev)[0];
                        const staffList = prev[firstWeekKey]?.staff || initialRotaData.staff;
                        return {
                            ...prev,
                            [currentWeekKey]: {
                                staff: staffList,
                                openingHours: initialRotaData.openingHours, // Always use default opening hours for a new week
                                shifts: []
                            }
                        }
                    });
                }
            }, [currentWeekKey, rotaData]);

            const handleDateChange = (days) => {
                setCurrentDate(prevDate => {
                    const newDate = new Date(prevDate);
                    newDate.setDate(newDate.getDate() + days);
                    return newDate;
                });
            };

            const handleAddShift = (day, department) => {
                setSelectedDay(day);
                setSelectedDepartment(department);
                setSelectedShift(null);
                setIsShiftModalOpen(true);
            };
            
            const handleEditShift = (shift) => {
                setSelectedDay(shift.day);
                setSelectedShift(shift);
                setIsShiftModalOpen(true);
            };

            const handleSaveShift = (shiftData) => {
                setRotaData(prev => {
                    const newRota = { ...prev };
                    const weekData = { ...(newRota[currentWeekKey] || { ...initialRotaData, shifts: [] }) };
                    
                    if (selectedShift) {
                        weekData.shifts = weekData.shifts.map(s => s.id === selectedShift.id ? { ...s, ...shiftData } : s);
                    } else {
                        const newShift = { ...shiftData, id: Date.now() };
                        weekData.shifts = [...weekData.shifts, newShift];
                    }
                    
                    newRota[currentWeekKey] = weekData;
                    return newRota;
                });
                setIsShiftModalOpen(false);
            };

            const handleDeleteShift = (shiftId) => {
                setRotaData(prev => {
                    const newRota = { ...prev };
                    const weekData = { ...newRota[currentWeekKey] };
                    weekData.shifts = weekData.shifts.filter(s => s.id !== shiftId);
                    newRota[currentWeekKey] = weekData;
                    return newRota;
                });
                setIsShiftModalOpen(false);
            };

            const handleCopyDayFromPrevWeek = (day) => {
                const prevWeekDate = new Date(currentDate);
                prevWeekDate.setDate(prevWeekDate.getDate() - 7);
                const prevWeekKey = formatDate(getWeekStartDate(prevWeekDate));

                if (rotaData[prevWeekKey] && rotaData[prevWeekKey].shifts) {
                    const shiftsToCopy = rotaData[prevWeekKey].shifts.filter(s => s.day === day);
                    setRotaData(prev => {
                        const newRota = { ...prev };
                        const weekData = { ...(newRota[currentWeekKey] || { ...initialRotaData, shifts: [] }) };
                        const otherDaysShifts = weekData.shifts.filter(s => s.day !== day);
                        const copiedShiftsWithNewIds = shiftsToCopy.map(s => ({...s, id: Date.now() + Math.random()}));
                        weekData.shifts = [...otherDaysShifts, ...copiedShiftsWithNewIds];
                        newRota[currentWeekKey] = weekData;
                        return newRota;
                    });
                } else {
                    alert("No data available for the previous week to copy.");
                }
            };
            
            const handleSaveStaff = (newStaffList) => {
                setRotaData(prev => {
                    const newRota = {...prev};
                    for(const weekKey in newRota){
                        if(newRota[weekKey]) {
                           newRota[weekKey].staff = newStaffList;
                        }
                    }
                    return newRota;
                });
                setIsStaffModalOpen(false);
            };
            
            const handleOpeningHoursChange = (day, type, value) => {
                setRotaData(prev => {
                    const newRota = { ...prev };
                    const weekData = { ...newRota[currentWeekKey] };
                    weekData.openingHours = { ...weekData.openingHours, [day]: { ...weekData.openingHours[day], [type]: value } };
                    newRota[currentWeekKey] = weekData;
                    return newRota;
                });
            };

            const downloadPdf = () => {
                const { jsPDF } = window.jspdf;
                const rotaContainer = document.getElementById('rota-container');
                const controlsContainer = document.getElementById('controls-container');
                
                if(controlsContainer) controlsContainer.style.display = 'none';

                html2canvas(rotaContainer, { scale: 2, useCORS: true }).then(canvas => {
                    const imgData = canvas.toDataURL('image/png');
                    const pdf = new jsPDF({
                        orientation: 'landscape',
                        unit: 'pt',
                        format: [canvas.width, canvas.height]
                    });
                    pdf.addImage(imgData, 'PNG', 0, 0, canvas.width, canvas.height);
                    pdf.save(`foresters-rota-${currentWeekKey}.pdf`);
                    
                    if(controlsContainer) controlsContainer.style.display = 'flex';
                });
            };

            const currentWeekData = rotaData[currentWeekKey] || { staff: [], openingHours: {}, shifts: [] };
            const { staff, openingHours, shifts } = currentWeekData;
            const daysOfWeek = ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"];

            const calculateHours = useCallback((startTime, endTime, breakMinutes) => {
                if (!startTime || !endTime) return 0;
                const start = new Date(`1970-01-01T${startTime}:00`);
                const end = new Date(`1970-01-01T${endTime}:00`);
                if (end <= start) { end.setDate(end.getDate() + 1); } // Handle overnight shifts
                const diffMs = end - start;
                return (diffMs / (1000 * 60 * 60)) - (breakMinutes / 60);
            }, []);

            const staffTotals = (staff || []).reduce((acc, s) => {
                acc[s.name] = (shifts || [])
                    .filter(shift => shift.staff === s.name)
                    .reduce((total, shift) => total + calculateHours(shift.startTime, shift.endTime, shift.breakMinutes), 0);
                return acc;
            }, {});
            
            const UnfilledHoursTally = ({ day, shifts, openingHours }) => {
                const dayHours = openingHours?.[day];
                if (!dayHours || !dayHours.open || !dayHours.close) return null;

                let openInMinutes = timeToMinutes(dayHours.open);
                let closeInMinutes = timeToMinutes(dayHours.close);

                if (closeInMinutes <= openInMinutes) {
                    closeInMinutes += 24 * 60;
                }

                const barShifts = (shifts || [])
                    .filter(s => s.day === day && s.department === 'Bar')
                    .map(s => {
                        let start = timeToMinutes(s.startTime);
                        let end = timeToMinutes(s.endTime);
                        if (end <= start) end += 24 * 60;
                        return { start, end };
                    })
                    .sort((a, b) => a.start - b.start);

                let lastCoveredTime = openInMinutes;
                const gaps = [];

                barShifts.forEach(shift => {
                    if (shift.start > lastCoveredTime) {
                        gaps.push({ start: lastCoveredTime, end: shift.start });
                    }
                    lastCoveredTime = Math.max(lastCoveredTime, shift.end);
                });

                if (lastCoveredTime < closeInMinutes) {
                    gaps.push({ start: lastCoveredTime, end: closeInMinutes });
                }

                if (gaps.length === 0) return null;

                const totalGapMinutes = gaps.reduce((total, gap) => total + (gap.end - gap.start), 0);
                const totalGapHours = totalGapMinutes / 60;

                if (totalGapHours <= 0) return null;

                const gapStrings = gaps.map(gap => `${minutesToTime(gap.start)} - ${minutesToTime(gap.end)}`);

                return (
                    <div className="mt-2 p-2 bg-red-100 border border-red-300 rounded-md text-red-800 text-xs">
                        <p className="font-bold">{totalGapHours.toFixed(2)} Bar Hours Unfilled</p>
                        <p>{gapStrings.join(', ')}</p>
                    </div>
                );
            };

            const ShiftList = ({ shifts, onEdit }) => {
                if (!shifts || shifts.length === 0) {
                    return <div className="flex-grow flex items-center justify-center"><p className="text-gray-400 italic text-sm">No shifts</p></div>;
                }
                return (
                    <div className="flex-grow space-y-1">
                        {shifts.map(shift => {
                            const staffMember = (staff || []).find(s => s.name === shift.staff);
                            const hours = calculateHours(shift.startTime, shift.endTime, shift.breakMinutes);
                            return (
                                <div key={shift.id} onClick={() => onEdit(shift)} className="flex justify-between items-center p-2 rounded-md cursor-pointer hover:bg-gray-200 transition-colors" style={{ borderLeft: `5px solid ${staffMember?.color || '#ccc'}`}}>
                                    <span className="font-semibold text-sm" style={{ color: staffMember?.color || '#333' }}>{shift.staff}</span>
                                    <span className="text-gray-600 text-xs">{shift.startTime}-{shift.endTime}</span>
                                    <span className="font-bold text-gray-800 text-xs">{hours.toFixed(1)} hrs</span>
                                </div>
                            );
                        })}
                    </div>
                );
            };

            const weekStartDate = getWeekStartDate(currentDate);

            return (
                <React.Fragment>
                    {isShiftModalOpen && <ShiftModal shift={selectedShift} day={selectedDay} staffList={staff} onSave={handleSaveShift} onClose={() => setIsShiftModalOpen(false)} onDelete={handleDeleteShift} department={selectedDepartment} />}
                    {isStaffModalOpen && <StaffModal staffList={staff || []} onSave={handleSaveStaff} onClose={() => setIsStaffModalOpen(false)} />}

                    <div id="rota-container" className="container mx-auto p-4 md:p-8 bg-white">
                        <header className="text-center mb-8">
                             <img src={logoSrc} alt="The Foresters Arms Logo" className="mx-auto mb-4 h-auto w-full max-w-xs" />
                            <div className="flex justify-center items-center space-x-2 sm:space-x-4 mt-4">
                                <button onClick={() => handleDateChange(-7)} className="bg-gray-200 text-gray-800 font-bold py-2 px-4 rounded-lg hover:bg-gray-300 transition-colors">&larr; Prev</button>
                                <p className="text-lg sm:text-xl text-gray-800 font-semibold w-48 sm:w-56 text-center">Week of {currentWeekKey}</p>
                                <button onClick={() => handleDateChange(7)} className="bg-gray-200 text-gray-800 font-bold py-2 px-4 rounded-lg hover:bg-gray-400 transition-colors">Next &rarr;</button>
                            </div>
                        </header>

                        <main className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-4">
                            {daysOfWeek.map((day, index) => {
                                const dayDate = new Date(weekStartDate);
                                dayDate.setDate(weekStartDate.getDate() + index);
                                const formattedDayDate = new Intl.DateTimeFormat('en-GB', { day: 'numeric', month: 'short' }).format(dayDate);

                                const dayOpeningHours = openingHours ? openingHours[day] : { open: '00:00', close: '00:00' };
                                const barShifts = (shifts || []).filter(s => s.day === day && s.department === 'Bar');
                                const kitchenShifts = (shifts || []).filter(s => s.day === day && s.department === 'Kitchen');
                                
                                return (
                                    <div key={day} className="bg-gray-50 p-4 rounded-lg shadow-md flex flex-col border border-gray-200">
                                        <div className="flex justify-between items-start">
                                            <h3 className="text-xl font-bold text-gray-700">{day} <span className="text-base font-normal text-gray-500">{formattedDayDate}</span></h3>
                                            <div className="text-xs text-gray-500 text-right">
                                                <input type="time" value={dayOpeningHours.open} onChange={(e) => handleOpeningHoursChange(day, 'open', e.target.value)} className="bg-transparent text-right w-20 p-0"/>
                                                <span>-</span>
                                                <input type="time" value={dayOpeningHours.close} onChange={(e) => handleOpeningHoursChange(day, 'close', e.target.value)} className="bg-transparent text-right w-20 p-0"/>
                                            </div>
                                        </div>
                                        <div className="flex-grow mt-4">
                                            <div className="mb-4">
                                                <div className="flex justify-between items-center border-b pb-1 mb-2">
                                                    <h4 className="font-semibold text-gray-500 text-sm">Bar</h4>
                                                    <button onClick={() => handleAddShift(day, 'Bar')} className="text-xs bg-blue-100 text-blue-700 font-bold h-6 w-6 rounded-full hover:bg-blue-200 flex items-center justify-center transition-transform hover:scale-110">+</button>
                                                </div>
                                                <div className="space-y-1 min-h-[60px] flex flex-col">
                                                    <ShiftList shifts={barShifts} onEdit={handleEditShift} />
                                                </div>
                                                <UnfilledHoursTally day={day} shifts={shifts} openingHours={openingHours} />
                                            </div>
                                            <div>
                                                <div className="flex justify-between items-center border-b pb-1 mb-2">
                                                    <h4 className="font-semibold text-gray-500 text-sm">Kitchen</h4>
                                                    <button onClick={() => handleAddShift(day, 'Kitchen')} className="text-xs bg-blue-100 text-blue-700 font-bold h-6 w-6 rounded-full hover:bg-blue-200 flex items-center justify-center transition-transform hover:scale-110">+</button>
                                                </div>
                                                <div className="space-y-1 min-h-[60px] flex flex-col">
                                                    <ShiftList shifts={kitchenShifts} onEdit={handleEditShift} />
                                                </div>
                                            </div>
                                        </div>
                                        <div className="flex justify-end items-center mt-4 pt-4 border-t border-gray-200">
                                             <button onClick={() => handleCopyDayFromPrevWeek(day)} className="text-xs bg-gray-200 text-gray-600 py-1 px-2 rounded hover:bg-gray-300">Copy Prev. Week</button>
                                        </div>
                                    </div>
                                );
                            })}
                        </main>

                        <footer className="mt-12 p-6 bg-white rounded-lg shadow-md">
                            <h2 className="text-2xl font-bold mb-4">Weekly Totals</h2>
                            <div className="grid grid-cols-2 md:grid-cols-4 gap-4">
                                {(staff || []).map(s => {
                                    const total = staffTotals[s.name] || 0;
                                    if (total > 0) {
                                        return (
                                            <div key={s.name} className="flex justify-between items-center text-lg p-2 rounded-md" style={{backgroundColor: `${s.color}20`}}>
                                                <span className="font-semibold">{s.name}</span>
                                                <span className="font-bold text-lg" style={{ color: s.color }}>{total.toFixed(2)} hrs</span>
                                            </div>
                                        );
                                    }
                                    return null;
                                })}
                            </div>
                        </footer>
                    </div>
                    
                    <div id="controls-container" className="container mx-auto px-4 md:px-8 my-6 flex justify-center items-center space-x-4">
                        <button onClick={() => setIsStaffModalOpen(true)} className="bg-blue-600 text-white font-bold py-3 px-6 rounded-lg shadow-lg hover:bg-blue-700 transition-transform hover:scale-105">
                            Manage Staff
                        </button>
                         <button onClick={downloadPdf} className="bg-green-600 text-white font-bold py-3 px-6 rounded-lg shadow-lg hover:bg-green-700 transition-transform hover:scale-105">
                            Download as PDF
                        </button>
                    </div>
                </React.Fragment>
            );
        };

        const container = document.getElementById('root');
        const root = ReactDOM.createRoot(container);
        root.render(<App />);
    </script>

</body>
</html>

