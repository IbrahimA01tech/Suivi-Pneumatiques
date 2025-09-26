<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Suivi des Pneumatiques</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Poppins:wght@400;500;600;700&display=swap');
        
        body {
            font-family: 'Poppins', sans-serif;
            background-color: #f4f6f9; /* Gris clair */
            color: #1a202c;
        }
        .container {
            max-width: 1100px;
            margin: auto;
            padding: 2rem;
        }
        .card {
            background-color: #ffffff;
            border-radius: 1.5rem;
            box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04);
            padding: 2.5rem;
        }
        .input-field {
            @apply w-full px-5 py-3 border border-gray-200 rounded-xl focus:outline-none focus:ring-3 focus:ring-blue-500 transition-all duration-300;
        }
        .btn {
            @apply px-8 py-3 rounded-full font-semibold transition-all duration-300 transform hover:scale-105 shadow-md;
        }
        .btn-primary {
            @apply bg-blue-600 text-white hover:bg-blue-700;
        }
        .btn-secondary {
            @apply bg-gray-200 text-gray-800 hover:bg-gray-300;
        }
        .btn-delete {
            @apply bg-red-500 text-white hover:bg-red-600;
        }
        .btn-update {
            @apply bg-green-500 text-white hover:bg-green-600;
        }
        .btn-gemini {
            @apply bg-purple-600 text-white hover:bg-purple-700;
        }
        .btn-import {
            @apply bg-blue-500 text-white hover:bg-blue-600;
        }
        .loader {
            border: 4px solid #f3f3f3;
            border-radius: 50%;
            border-top: 4px solid #3498db;
            width: 24px;
            height: 24px;
            -webkit-animation: spin 2s linear infinite;
            animation: spin 2s linear infinite;
        }
        @-webkit-keyframes spin {
            0% { -webkit-transform: rotate(0deg); }
            100% { -webkit-transform: rotate(360deg); }
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
        .modal {
            display: none;
            position: fixed;
            z-index: 100;
            left: 0;
            top: 0;
            width: 100%;
            height: 100%;
            overflow: auto;
            background-color: rgb(0,0,0);
            background-color: rgba(0,0,0,0.4);
            justify-content: center;
            align-items: center;
        }
        .modal-content {
            background-color: #fefefe;
            padding: 20px;
            border-radius: 1.5rem;
            width: 80%;
            max-width: 500px;
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
        }
    </style>
</head>
<body class="bg-gray-50 p-4 min-h-screen flex flex-col items-center">

    <div id="message-box" class="fixed top-5 left-1/2 -translate-x-1/2 z-50 p-4 rounded-full shadow-lg hidden transition-opacity duration-300"></div>
    <div id="loading-modal" class="modal">
        <div class="modal-content flex flex-col items-center p-8">
            <div class="loader"></div>
            <p class="mt-4 text-lg font-semibold text-gray-700">Génération du rapport...</p>
        </div>
    </div>
    <div id="confirm-modal" class="modal">
        <div class="modal-content text-center">
            <p id="confirm-message" class="text-lg font-medium mb-4">Êtes-vous sûr de vouloir supprimer ce pneumatique ?</p>
            <div class="flex justify-center gap-4">
                <button id="confirm-yes" class="btn btn-delete">Oui</button>
                <button id="confirm-no" class="btn btn-secondary">Non</button>
            </div>
        </div>
    </div>

    <div class="container space-y-12">
        <header class="text-center mb-6">
            <h1 class="text-5xl md:text-6xl font-bold text-blue-800 tracking-tight leading-tight">
                Gestion des Pneumatiques
            </h1>
            <p class="mt-2 text-gray-600 text-lg">Suivez l'état des pneus de votre flotte en temps réel.</p>
        </header>

        <!-- Formulaire d'ajout de pneumatique -->
        <div class="card">
            <h2 class="text-2xl font-bold mb-6 text-gray-800 flex items-center gap-3">
                <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" class="w-8 h-8 text-blue-600">
                  <path d="M12 2.25c-5.385 0-9.75 4.365-9.75 9.75s4.365 9.75 9.75 9.75 9.75-4.365 9.75-9.75S17.385 2.25 12 2.25ZM12.75 9a.75.75 0 0 0-1.5 0v2.25H9a.75.75 0 0 0 0 1.5h2.25V15a.75.75 0 0 0 1.5 0v-2.25H15a.75.75 0 0 0 0-1.5h-2.25V9Z" />
                </svg>
                Ajouter / Modifier un Pneumatique
            </h2>
            <form id="tire-form" class="grid grid-cols-1 md:grid-cols-2 gap-6">
                <input type="hidden" id="tire-id">
                <div class="space-y-1">
                    <label for="reference" class="text-sm font-medium text-gray-700">Référence</label>
                    <input type="text" id="reference" placeholder="Ex: Pneu-A123" class="input-field" required>
                </div>
                <div class="space-y-1">
                    <label for="dimensions" class="text-sm font-medium text-gray-700">Dimensions</label>
                    <input type="text" id="dimensions" placeholder="Ex: 205/55R16" class="input-field" required>
                </div>
                <div class="space-y-1">
                    <label for="brand" class="text-sm font-medium text-gray-700">Marque</label>
                    <input type="text" id="brand" placeholder="Ex: Michelin" class="input-field" required>
                </div>
                <div class="space-y-1">
                    <label for="wear" class="text-sm font-medium text-gray-700">Usure de la gomme (%)</label>
                    <input type="number" id="wear" min="0" max="100" placeholder="Ex: 25" class="input-field" required>
                </div>
                <div class="space-y-1">
                    <label for="position" class="text-sm font-medium text-gray-700">Position sur essieu</label>
                    <input type="text" id="position" placeholder="Ex: Essieu avant droit" class="input-field" required>
                </div>
                <div class="space-y-1">
                    <label for="km-counter" class="text-sm font-medium text-gray-700">Compteur kilométrique (km)</label>
                    <input type="number" id="km-counter" min="0" placeholder="Ex: 50000" class="input-field" required>
                </div>
                <div class="space-y-1">
                    <label for="hour-counter" class="text-sm font-medium text-gray-700">Compteur horaire (heures)</label>
                    <input type="number" id="hour-counter" min="0" placeholder="Ex: 1250" class="input-field" required>
                </div>
                <div class="space-y-1">
                    <label for="pressure" class="text-sm font-medium text-gray-700">Pression (BARS)</label>
                    <input type="number" id="pressure" min="0" placeholder="Ex: 9" class="input-field" required>
                </div>
                <div class="space-y-1">
                    <label for="equipment" class="text-sm font-medium text-gray-700">Équipement</label>
                    <input type="text" id="equipment" placeholder="Ex: Camion 123" class="input-field" required>
                </div>
                <div class="md:col-span-2 flex justify-end gap-3 mt-4">
                    <button type="submit" class="btn btn-primary">
                        Enregistrer
                    </button>
                    <button type="reset" class="btn btn-secondary" id="cancel-btn">
                        Annuler
                    </button>
                </div>
            </form>
        </div>

        <!-- Section du rapport et de la liste des pneumatiques -->
        <div class="grid grid-cols-1 lg:grid-cols-2 gap-12">
            <!-- Rapport généré par Gemini -->
            <div class="card h-full" id="gemini-report-card">
                <h2 class="text-2xl font-bold mb-4 text-gray-800 flex items-center gap-3">
                    <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" class="w-8 h-8 text-purple-600">
                      <path d="M11.47 3.84a.75.75 0 1 0-1.22-.886L9.61 3.52l-1.39-1.296a.75.75 0 0 0-1.17 0l-1.39 1.296-1.076-.201a.75.75 0 0 0-.886.525L5.053 5.31l-1.296 1.39a.75.75 0 0 0 0 1.17l1.296 1.39-.201 1.076a.75.75 0 0 0 .525.886l1.785.334 1.39 1.296a.75.75 0 0 0 1.17 0l1.39-1.296 1.785-.334a.75.75 0 0 0 .525-.886l-.201-1.076 1.296-1.39a.75.75 0 0 0 0-1.17l-1.296-1.39.334-1.785ZM12 6.75a2.25 2.25 0 1 1-4.5 0 2.25 2.25 0 0 1 4.5 0ZM14.25 17.5a.75.75 0 0 0 0-1.5H12a2.25 2.25 0 0 1-2.25-2.25v-2.25a.75.75 0 0 0-1.5 0v2.25A3.75 3.75 0 0 0 12 16h2.25v1.5ZM12 17.5a.75.75 0 0 0 0 1.5h.75a2.25 2.25 0 0 1 2.25 2.25v2.25a.75.75 0 0 0 1.5 0v-2.25a3.75 3.75 0 0 0-3.75-3.75H12v1.5ZM12 19a.75.75 0 0 0 0 1.5h.75a2.25 2.25 0 0 1 2.25 2.25v.75a.75.75 0 0 0 1.5 0v-.75A3.75 3.75 0 0 0 12.75 19H12Z" />
                    </svg>
                    Rapport d'analyse ✨
                </h2>
                <div id="gemini-report-content" class="prose max-w-none text-gray-700">
                    <p class="text-center text-gray-500">
                        Les rapports générés par l'IA apparaîtront ici.
                    </p>
                </div>
            </div>

            <!-- Liste des pneumatiques -->
            <div class="card h-full">
                <h2 class="text-2xl font-bold mb-4 text-gray-800 flex justify-between items-center">
                    <span class="flex items-center gap-3">
                        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" class="w-8 h-8 text-blue-600">
                          <path d="M7.5 3.375c0-.68.56-1.238 1.238-1.238h6.474c.678 0 1.238.558 1.238 1.238v3.626a.75.75 0 0 1-1.5 0V4.5h-6v2.502a.75.75 0 0 1-1.5 0V3.375Z" />
                          <path fill-rule="evenodd" d="M3 8.375a.75.75 0 0 1 .75-.75h16.5a.75.75 0 0 1 .75.75v11.25c0 1.63-1.32 2.95-2.95 2.95H5.95c-1.63 0-2.95-1.32-2.95-2.95V8.375ZM5.95 21a1.45 1.45 0 0 1-1.45-1.45v-10.3h16.5v10.3a1.45 1.45 0 0 1-1.45 1.45H5.95Z" clip-rule="evenodd" />
                        </svg>
                        Liste des pneumatiques
                    </span>
                    <button id="generate-fleet-report-btn" class="btn btn-gemini flex items-center gap-2">
                        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" class="w-5 h-5">
                          <path d="M11.47 3.84a.75.75 0 1 0-1.22-.886L9.61 3.52l-1.39-1.296a.75.75 0 0 0-1.17 0l-1.39 1.296-1.076-.201a.75.75 0 0 0-.886.525L5.053 5.31l-1.296 1.39a.75.75 0 0 0 0 1.17l1.296 1.39-.201 1.076a.75.75 0 0 0 .525.886l1.785.334 1.39 1.296a.75.75 0 0 0 1.17 0l1.39-1.296 1.785-.334a.75.75 0 0 0 .525-.886l-.201-1.076 1.296-1.39a.75.75 0 0 0 0-1.17l-1.296-1.39.334-1.785ZM12 6.75a2.25 2.25 0 1 1-4.5 0 2.25 2.25 0 0 1 4.5 0ZM14.25 17.5a.75.75 0 0 0 0-1.5H12a2.25 2.25 0 0 1-2.25-2.25v-2.25a.75.75 0 0 0-1.5 0v2.25A3.75 3.75 0 0 0 12 16h2.25v1.5ZM12 17.5a.75.75 0 0 0 0 1.5h.75a2.25 2.25 0 0 1 2.25 2.25v2.25a.75.75 0 0 0 1.5 0v-2.25a3.75 3.75 0 0 0-3.75-3.75H12v1.5ZM12 19a.75.75 0 0 0 0 1.5h.75a2.25 2.25 0 0 1 2.25 2.25v.75a.75.75 0 0 0 1.5 0v-.75A3.75 3.75 0 0 0 12.75 19H12Z" />
                        </svg>
                        Rapport du parc
                    </button>
                </h2>
                <div id="tire-list" class="space-y-4">
                    <p class="text-gray-500 text-center" id="loading-message">Chargement des données...</p>
                    <!-- Les cartes de pneus seront insérées ici par JavaScript -->
                </div>
            </div>
        </div>
        
        <!-- Importation de fichier -->
        <div class="card">
            <h2 class="text-2xl font-bold mb-4 text-gray-800 flex items-center gap-3">
                <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" class="w-8 h-8 text-blue-600">
                  <path fill-rule="evenodd" d="M12 2.25a.75.75 0 0 1 .75.75v11.69l3.22-3.22a.75.75 0 1 1 1.06 1.06l-4.5 4.5a.75.75 0 0 1-1.06 0l-4.5-4.5a.75.75 0 1 1 1.06-1.06l3.22 3.22V3a.75.75 0 0 1 .75-.75ZM9 18a.75.75 0 0 0-1.5 0v.75A2.25 2.25 0 0 0 9.75 21h4.5A2.25 2.25 0 0 0 16.5 18.75V18a.75.75 0 0 0-1.5 0v.75a.75.75 0 0 1-.75.75h-3a.75.75 0 0 1-.75-.75V18Z" clip-rule="evenodd" />
                </svg>
                Importer depuis un Fichier XLSX
            </h2>
            <div class="space-y-4">
                <p class="text-sm text-gray-600">Sélectionnez un fichier XLSX avec vos données de pneumatiques. Les données existantes seront effacées avant l'importation.</p>
                <input type="file" id="xlsx-file-input" class="block w-full text-sm text-gray-500
                    file:mr-4 file:py-2 file:px-4
                    file:rounded-full file:border-0
                    file:text-sm file:font-semibold
                    file:bg-blue-50 file:text-blue-700
                    hover:file:bg-blue-100 cursor-pointer" accept=".xlsx, .xls">
                <button id="import-btn" class="btn btn-import w-full" disabled>Importer les données</button>
            </div>
        </div>
        
        <p class="mt-4 text-sm text-gray-500 text-center" id="user-id-display"></p>
    </div>

    <!-- Firebase CDN -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, collection, onSnapshot, doc, addDoc, updateDoc, deleteDoc, query, where, writeBatch, setDoc, getDocs } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        import { setLogLevel } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // Inclure les bibliothèques tierces via CDN
        const scriptXLSX = document.createElement('script');
        scriptXLSX.src = "https://cdn.sheetjs.com/xlsx-0.20.2/package/dist/xlsx.full.min.js";
        document.head.appendChild(scriptXLSX);

        const scriptDOMPurify = document.createElement('script');
        scriptDOMPurify.src = "https://cdn.jsdelivr.net/npm/dompurify@3.0.6/dist/purify.min.js";
        document.head.appendChild(scriptDOMPurify);

        // Définir le niveau de log pour un débogage facile
        setLogLevel('debug');

        // Récupérer les variables globales pour la configuration et l'authentification
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        const firebaseConfig = JSON.parse(typeof __firebase_config !== 'undefined' ? __firebase_config : '{}');
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

        // Configuration de l'API Gemini
        const API_KEY = "";
        const API_URL = "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent?key=" + API_KEY;
        const RETRY_DELAY = 1000;
        const MAX_RETRIES = 5;

        // Éléments du DOM
        const tireForm = document.getElementById('tire-form');
        const tireList = document.getElementById('tire-list');
        const tireIdInput = document.getElementById('tire-id');
        const messageBox = document.getElementById('message-box');
        const loadingMessage = document.getElementById('loading-message');
        const userIdDisplay = document.getElementById('user-id-display');
        const generateFleetReportBtn = document.getElementById('generate-fleet-report-btn');
        const geminiReportContent = document.getElementById('gemini-report-content');
        const loadingModal = document.getElementById('loading-modal');
        const confirmModal = document.getElementById('confirm-modal');
        const confirmMessage = document.getElementById('confirm-message');
        const confirmYesBtn = document.getElementById('confirm-yes');
        const confirmNoBtn = document.getElementById('confirm-no');
        const xlsxFileInput = document.getElementById('xlsx-file-input');
        const importBtn = document.getElementById('import-btn');

        let db;
        let auth;
        let userId;
        let allTires = [];
        let confirmPromiseResolver;
        let importedTireData = [];

        // Fonction pour nettoyer et sécuriser les chaînes de caractères
        function sanitizeInput(input) {
            return DOMPurify.sanitize(input, { USE_PROFILES: { html: false } });
        }

        // Fonction pour afficher un message à l'utilisateur
        function showMessage(message, type = 'success') {
            messageBox.textContent = message;
            messageBox.style.opacity = '1';
            messageBox.classList.remove('hidden', 'bg-green-500', 'bg-red-500', 'bg-yellow-500');
            if (type === 'success') {
                messageBox.classList.add('bg-green-500', 'text-white');
            } else if (type === 'error') {
                messageBox.classList.add('bg-red-500', 'text-white');
            } else {
                 messageBox.classList.add('bg-yellow-500', 'text-white');
            }
            setTimeout(() => {
                messageBox.style.opacity = '0';
            }, 3000);
        }

        // Fonction pour afficher une boîte de dialogue de confirmation personnalisée
        function showConfirm(message) {
            confirmMessage.textContent = message;
            confirmModal.style.display = 'flex';
            return new Promise((resolve) => {
                confirmPromiseResolver = resolve;
            });
        }
        confirmYesBtn.addEventListener('click', () => {
            confirmModal.style.display = 'none';
            confirmPromiseResolver(true);
        });
        confirmNoBtn.addEventListener('click', () => {
            confirmModal.style.display = 'none';
            confirmPromiseResolver(false);
        });


        // Fonction pour réinitialiser le formulaire
        function resetForm() {
            tireForm.reset();
            tireIdInput.value = '';
            document.querySelector('#tire-form button[type="submit"]').textContent = 'Enregistrer';
        }

        // Écouteur d'événements pour le bouton d'annulation
        document.getElementById('cancel-btn').addEventListener('click', (e) => {
            e.preventDefault();
            resetForm();
        });

        // Fonction pour créer une carte de pneu
        function createTireCard(tire) {
            const card = document.createElement('div');
            const sanitizedReference = sanitizeInput(tire.reference);
            const sanitizedDimensions = sanitizeInput(tire.dimensions);
            const sanitizedBrand = sanitizeInput(tire.brand);
            const sanitizedPosition = sanitizeInput(tire.position);
            const sanitizedEquipment = sanitizeInput(tire.equipment);
            
            card.className = 'card border-l-4 border-blue-400 flex flex-col md:flex-row justify-between items-start md:items-center space-y-4 md:space-y-0';
            card.dataset.id = tire.id;

            // Déterminer la couleur de la barre d'usure
            let wearColor = 'bg-green-500';
            if (tire.wear > 50) {
                wearColor = 'bg-yellow-500';
            }
            if (tire.wear > 80) {
                wearColor = 'bg-red-500';
            }

            // Déterminer la couleur de l'icône de pression
            let pressureColor = 'text-green-500';
            if (tire.pressure < 6) {
                pressureColor = 'text-red-500';
            }

            card.innerHTML = `
                <div class="flex-grow space-y-2">
                    <h3 class="text-xl font-bold text-gray-900">${sanitizedReference}</h3>
                    <p class="text-gray-600"><span class="font-semibold">Dimensions:</span> ${sanitizedDimensions}</p>
                    <p class="text-gray-600"><span class="font-semibold">Marque:</span> ${sanitizedBrand}</p>
                    <p class="text-gray-600 flex items-center gap-2"><span class="font-semibold">Position:</span> ${sanitizedPosition}</p>
                    <p class="text-gray-600 flex items-center gap-2"><span class="font-semibold">Équipement:</span> ${sanitizedEquipment}</p>
                    
                    <div class="flex items-center gap-4 pt-2">
                        <div class="flex items-center gap-2 text-gray-600 font-medium">
                            <span class="font-semibold">Usure:</span>
                            <div class="w-24 bg-gray-200 rounded-full h-2.5">
                                <div class="${wearColor} h-2.5 rounded-full" style="width: ${tire.wear}%;"></div>
                            </div>
                            <span class="text-sm">${tire.wear}%</span>
                        </div>
                        <div class="flex items-center gap-2 text-gray-600 font-medium">
                            <span class="font-semibold">Pression:</span>
                            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" class="w-6 h-6 ${pressureColor}">
                              <path d="M11.47 3.84a.75.75 0 1 0-1.22-.886L9.61 3.52l-1.39-1.296a.75.75 0 0 0-1.17 0l-1.39 1.296-1.076-.201a.75.75 0 0 0-.886.525L5.053 5.31l-1.296 1.39a.75.75 0 0 0 0 1.17l1.296 1.39-.201 1.076a.75.75 0 0 0 .525.886l1.785.334 1.39 1.296a.75.75 0 0 0 1.17 0l1.39-1.296 1.785-.334a.75.75 0 0 0 .525-.886l-.201-1.076 1.296-1.39a.75.75 0 0 0 0-1.17l-1.296-1.39.334-1.785ZM12 6.75a2.25 2.25 0 1 1-4.5 0 2.25 2.25 0 0 1 4.5 0ZM14.25 17.5a.75.75 0 0 0 0-1.5H12a2.25 2.25 0 0 1-2.25-2.25v-2.25a.75.75 0 0 0-1.5 0v2.25A3.75 3.75 0 0 0 12 16h2.25v1.5ZM12 17.5a.75.75 0 0 0 0 1.5h.75a2.25 2.25 0 0 1 2.25 2.25v2.25a.75.75 0 0 0 1.5 0v-2.25a3.75 3.75 0 0 0-3.75-3.75H12v1.5ZM12 19a.75.75 0 0 0 0 1.5h.75a2.25 2.25 0 0 1 2.25 2.25v.75a.75.75 0 0 0 1.5 0v-.75A3.75 3.75 0 0 0 12.75 19H12Z" />
                            </svg>
                            <span class="font-bold text-lg">${tire.pressure} BARS</span>
                        </div>
                    </div>
                </div>
                <div class="flex flex-col md:flex-row space-x-0 md:space-x-2 space-y-2 md:space-y-0 mt-2 md:mt-0">
                    <button class="btn btn-update flex items-center gap-1" data-action="edit">
                        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" class="w-5 h-5">
                          <path d="M21.731 2.269a2.625 2.625 0 0 0-3.712 0l-1.157 1.157 3.712 3.712 1.157-1.157a2.625 2.625 0 0 0 0-3.712ZM19.513 8.199l-3.712-3.712-8.4 8.4a5.25 5.25 0 0 0-1.332 2.214l-.5 2.149a.75.75 0 0 0 .933.933l2.149-.5a5.25 5.25 0 0 0 2.214-1.332l8.4-8.4Z" />
                          <path d="M15 17.25h.007v.008H15v-.008ZM12.75 18h.007v.008H12.75V18Z" />
                        </svg>
                        Modifier
                    </button>
                    <button class="btn btn-delete flex items-center gap-1" data-action="delete">
                        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" class="w-5 h-5">
                          <path fill-rule="evenodd" d="M16.5 4.478v.227a48.842 48.842 0 0 1 3.868.513A2.25 2.25 0 0 1 21 7.756v10.153a2.25 2.25 0 0 1-2.25 2.25H5.25a2.25 2.25 0 0 1-2.25-2.25V7.756A2.25 2.25 0 0 1 4.632 5.218a48.842 48.842 0 0 1 3.868-.513V4.478c0-.776.541-1.42 1.272-1.554A48.89 48.89 0 0 1 12 2.25c.582 0 1.158.068 1.728.19A1.5 1.5 0 0 1 15 4.478Z" clip-rule="evenodd" />
                          <path d="M12.75 15.375a.75.75 0 0 0-1.5 0v2.25a.75.75 0 0 0 1.5 0v-2.25ZM12.75 12a.75.75 0 0 0-1.5 0v2.25a.75.75 0 0 0 1.5 0V12ZM9.75 9a.75.75 0 0 0-1.5 0v.75a.75.75 0 0 0 1.5 0V9ZM15.75 9a.75.75 0 0 0-1.5 0v.75a.75.75 0 0 0 1.5 0V9Z" />
                        </svg>
                        Supprimer
                    </button>
                </div>
            `;
            return card;
        }

        // Fonction pour appeler l'API Gemini avec une logique de réessai exponentiel
        async function callGeminiAPI(prompt, retries = 0) {
            const payload = {
                contents: [{ parts: [{ text: prompt }] }],
            };
            
            try {
                const response = await fetch(API_URL, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });

                if (!response.ok) {
                    if (response.status === 429 && retries < MAX_RETRIES) {
                        const delay = RETRY_DELAY * Math.pow(2, retries);
                        await new Promise(res => setTimeout(res, delay));
                        return callGeminiAPI(prompt, retries + 1);
                    }
                    throw new Error(`API Error: ${response.status} ${response.statusText}`);
                }

                const result = await response.json();
                const candidate = result.candidates?.[0];
                if (candidate && candidate.content?.parts?.[0]?.text) {
                    return candidate.content.parts[0].text;
                } else {
                    throw new Error("Invalid response from API");
                }
            } catch (error) {
                console.error("Error calling Gemini API:", error);
                return "Une erreur s'est produite lors de la génération du rapport. Veuillez réessayer plus tard.";
            }
        }

        // Fonction pour générer un rapport sur l'ensemble du parc de pneumatiques
        async function generateFleetReport() {
            loadingModal.style.display = 'flex';
            geminiReportContent.innerHTML = '<p class="text-center text-gray-500">Génération du rapport en cours...</p>';

            const tiresData = allTires.map(tire => ({
                reference: tire.reference,
                marque: tire.brand,
                dimensions: tire.dimensions,
                usure: tire.wear,
                position: tire.position,
                kilometrage: tire.kmCounter,
                heures: tire.hourCounter,
                pression: tire.pressure,
                equipment: tire.equipment
            }));

            if (tiresData.length === 0) {
                loadingModal.style.display = 'none';
                geminiReportContent.innerHTML = '<p class="text-center text-gray-500">Il n\'y a pas de pneumatiques à analyser.</p>';
                return;
            }

            let tableHTML = `<div class="mb-6"><h3 class="text-xl font-semibold mb-2">Synthèse visuelle du parc</h3>`;
            tableHTML += `<div class="overflow-x-auto"><table class="min-w-full divide-y divide-gray-200 rounded-lg shadow-sm">
                <thead class="bg-gray-50">
                    <tr>
                        <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Référence</th>
                        <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Équipement</th>
                        <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Usure (%)</th>
                        <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Pression (BARS)</th>
                    </tr>
                </thead>
                <tbody class="bg-white divide-y divide-gray-200">`;
            tiresData.forEach(tire => {
                const sanitizedTire = {
                    reference: sanitizeInput(tire.reference),
                    equipment: sanitizeInput(tire.equipment),
                    wear: tire.wear,
                    pressure: tire.pressure
                };
                tableHTML += `
                    <tr>
                        <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">${sanitizedTire.reference}</td>
                        <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">${sanitizedTire.equipment}</td>
                        <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">${sanitizedTire.wear}%</td>
                        <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">${sanitizedTire.pressure}</td>
                    </tr>
                `;
            });
            tableHTML += `</tbody></table></div></div>`;

            const prompt = `En tant qu'analyste de flotte, générez un résumé très concis et en quelques lignes de l'état général des pneumatiques. Mettez en évidence les problèmes les plus urgents (par exemple, les pneus les plus usés ou ceux ayant une pression inférieure à 6 bars). La pression optimale est de 9 bars. Donnez des recommandations prioritaires.

            Données des pneumatiques :
            ${JSON.stringify(tiresData, null, 2)}

            Résumé :`;

            const report = await callGeminiAPI(prompt);
            const sanitizedReport = DOMPurify.sanitize(report);
            geminiReportContent.innerHTML = tableHTML + `<div class="p-4 bg-gray-50 rounded-lg text-gray-700">${sanitizedReport}</div>`;
            loadingModal.style.display = 'none';
        }

        // Initialisation de l'application Firebase
        window.onload = async () => {
            try {
                const app = initializeApp(firebaseConfig);
                db = getFirestore(app);
                auth = getAuth(app);

                onAuthStateChanged(auth, async (user) => {
                    if (user) {
                        userId = user.uid;
                        userIdDisplay.textContent = `ID utilisateur : ${userId}`;
                        setupRealtimeListener();
                    } else {
                         try {
                            if (initialAuthToken) {
                                await signInWithCustomToken(auth, initialAuthToken);
                            } else {
                                await signInAnonymously(auth);
                            }
                        } catch (error) {
                            console.error("Erreur d'authentification:", error);
                            showMessage("Erreur d'authentification. Veuillez réessayer.", 'error');
                        }
                    }
                });

            } catch (e) {
                console.error("Erreur lors de l'initialisation de Firebase: ", e);
                showMessage("Erreur de connexion à la base de données. Vérifiez la configuration.", 'error');
            }
        };

        // Configuration de l'écouteur en temps réel pour Firestore
        function setupRealtimeListener() {
            if (!db) {
                console.error("La base de données n'est pas initialisée.");
                return;
            }

            const tiresCollectionRef = collection(db, `artifacts/${appId}/public/data/pneus`);

            onSnapshot(tiresCollectionRef, (querySnapshot) => {
                tireList.innerHTML = '';
                allTires = [];
                if (querySnapshot.empty) {
                    tireList.innerHTML = '<p class="text-center text-gray-500">Aucun pneumatique trouvé. Ajoutez-en un !</p>';
                } else {
                    querySnapshot.forEach((doc) => {
                        const tire = { id: doc.id, ...doc.data() };
                        allTires.push(tire);
                        const tireCard = createTireCard(tire);
                        tireList.appendChild(tireCard);
                    });
                }
                loadingMessage.style.display = 'none';
            }, (error) => {
                console.error("Erreur de l'écouteur en temps réel: ", error);
                showMessage("Erreur de synchronisation des données.", 'error');
                loadingMessage.textContent = "Impossible de charger les données.";
            });
        }

        // Soumission du formulaire (ajout ou modification)
        tireForm.addEventListener('submit', async (e) => {
            e.preventDefault();

            const reference = sanitizeInput(document.getElementById('reference').value);
            const dimensions = sanitizeInput(document.getElementById('dimensions').value);
            const brand = sanitizeInput(document.getElementById('brand').value);
            const wear = parseFloat(document.getElementById('wear').value);
            const position = sanitizeInput(document.getElementById('position').value);
            const kmCounter = parseInt(document.getElementById('km-counter').value, 10);
            const hourCounter = parseInt(document.getElementById('hour-counter').value, 10);
            const pressure = parseFloat(document.getElementById('pressure').value);
            const equipment = sanitizeInput(document.getElementById('equipment').value);

            if (wear < 0 || wear > 100 || kmCounter < 0 || hourCounter < 0 || pressure < 0) {
                 showMessage('Les valeurs numériques doivent être positives.', 'error');
                 return;
            }

            const tireData = {
                reference,
                dimensions,
                brand,
                wear,
                position,
                kmCounter,
                hourCounter,
                pressure,
                equipment,
                lastUpdated: new Date().toISOString()
            };

            const tireId = tireIdInput.value;

            try {
                if (tireId) {
                    const tireRef = doc(db, `artifacts/${appId}/public/data/pneus`, tireId);
                    await updateDoc(tireRef, tireData);
                    showMessage('Pneumatique modifié avec succès !');
                } else {
                    await addDoc(collection(db, `artifacts/${appId}/public/data/pneus`), tireData);
                    showMessage('Pneumatique ajouté avec succès !');
                }
                resetForm();
            } catch (error) {
                console.error("Erreur lors de l'enregistrement: ", error);
                showMessage('Erreur lors de l\'enregistrement des données. Veuillez réessayer.', 'error');
            }
        });

        // Gestion des actions (modifier, supprimer, et analyse Gemini)
        tireList.addEventListener('click', async (e) => {
            const action = e.target.dataset.action;
            const card = e.target.closest('.card');
            if (!card) return;

            const tireId = card.dataset.id;

            if (action === 'delete') {
                const isConfirmed = await showConfirm('Êtes-vous sûr de vouloir supprimer ce pneumatique ?');
                if (!isConfirmed) return;

                try {
                    const tireRef = doc(db, `artifacts/${appId}/public/data/pneus`, tireId);
                    await deleteDoc(tireRef);
                    showMessage('Pneumatique supprimé avec succès.');
                } catch (error) {
                    console.error("Erreur lors de la suppression: ", error);
                    showMessage('Erreur lors de la suppression. Veuillez réessayer.', 'error');
                }
            }

            if (action === 'edit') {
                const tireToEdit = allTires.find(t => t.id === tireId);
                if (tireToEdit) {
                    tireIdInput.value = tireId;
                    document.getElementById('reference').value = tireToEdit.reference;
                    document.getElementById('dimensions').value = tireToEdit.dimensions;
                    document.getElementById('brand').value = tireToEdit.brand;
                    document.getElementById('wear').value = tireToEdit.wear;
                    document.getElementById('position').value = tireToEdit.position;
                    document.getElementById('km-counter').value = tireToEdit.kmCounter;
                    document.getElementById('hour-counter').value = tireToEdit.hourCounter;
                    document.getElementById('pressure').value = tireToEdit.pressure;
                    document.getElementById('equipment').value = tireToEdit.equipment;
                    document.querySelector('#tire-form button[type="submit"]').textContent = 'Mettre à jour';
                }
            }
        });

        generateFleetReportBtn.addEventListener('click', () => {
            generateFleetReport();
        });

        // Logique d'importation de fichier XLSX
        xlsxFileInput.addEventListener('change', (event) => {
            if (event.target.files.length > 0) {
                importBtn.disabled = false;
            } else {
                importBtn.disabled = true;
            }
        });

        importBtn.addEventListener('click', () => {
            const file = xlsxFileInput.files[0];
            if (file) {
                const reader = new FileReader();
                reader.onload = async (e) => {
                    const data = new Uint8Array(e.target.result);
                    const workbook = XLSX.read(data, { type: 'array' });
                    const worksheet = workbook.Sheets['SUIVI'];
                    if (!worksheet) {
                        showMessage('La feuille de calcul "SUIVI" est introuvable dans le fichier.', 'error');
                        return;
                    }

                    const jsonData = XLSX.utils.sheet_to_json(worksheet, { header: 1 });
                    const parsedData = parseXLSXData(jsonData);

                    if (parsedData.length > 0) {
                        const isConfirmed = await showConfirm(`Êtes-vous sûr de vouloir importer ${parsedData.length} pneumatiques ? Toutes les données actuelles seront effacées.`);
                        if (!isConfirmed) return;
                        await importDataToFirestore(parsedData);
                    } else {
                        showMessage('Aucune donnée valide n\'a été trouvée dans le fichier.', 'error');
                    }
                };
                reader.readAsArrayBuffer(file);
            }
        });

        function parseXLSXData(jsonData) {
            const data = [];
            const headerRowIndex = jsonData.findIndex(row => row.includes('N°SERIE'));
            if (headerRowIndex === -1) {
                showMessage('Le format du fichier est incorrect. L\'en-tête "N°SERIE" est introuvable.', 'error');
                return [];
            }
            const headers = jsonData[headerRowIndex];
            const dataRows = jsonData.slice(headerRowIndex + 1);

            dataRows.forEach(row => {
                if (row.length === 0) return;

                const tire = {};
                headers.forEach((header, i) => {
                    tire[header] = row[i];
                });

                const dimensions = tire['Dimension'] || '';
                const treadDepth = parseFloat(tire['Gomme (mm)']) || 0;
                let wearPercentage;
                const minTreadDepth = 8;

                if (dimensions.includes('13R22.5')) {
                    const newTreadDepth = 25;
                    wearPercentage = Math.min(100, Math.max(0, Math.round(((treadDepth - minTreadDepth) / (newTreadDepth - minTreadDepth)) * 100)));
                } else if (dimensions.includes('12.00R24')) {
                    const newTreadDepth = 15;
                    wearPercentage = Math.min(100, Math.max(0, Math.round(((treadDepth - minTreadDepth) / (newTreadDepth - minTreadDepth)) * 100)));
                } else {
                    const newTreadDepth = 17;
                    wearPercentage = Math.min(100, Math.max(0, Math.round(((treadDepth - minTreadDepth) / (newTreadDepth - minTreadDepth)) * 100)));
                }

                const mappedTire = {
                    reference: sanitizeInput(tire['N°SERIE'] || ''),
                    dimensions: sanitizeInput(dimensions),
                    brand: sanitizeInput(tire['Marque'] || ''),
                    equipment: sanitizeInput(tire['Camion Engin'] || ''),
                    hourCounter: parseInt(tire['Compteur (h )'], 10) || 0,
                    kmCounter: parseInt(tire['Compteur (km)'], 10) || 0,
                    position: sanitizeInput(tire['Position'] || ''),
                    wear: wearPercentage,
                    pressure: parseFloat(tire['Pression (Bar)']) || 0,
                    lastUpdated: new Date().toISOString()
                };
                data.push(mappedTire);
            });
            return data;
        }

        async function importDataToFirestore(data) {
            loadingModal.style.display = 'flex';
            const batch = writeBatch(db);
            const tiresCollectionRef = collection(db, `artifacts/${appId}/public/data/pneus`);

            try {
                const existingDocs = await getDocs(tiresCollectionRef);
                existingDocs.forEach(doc => {
                    batch.delete(doc.ref);
                });

                data.forEach(tire => {
                    const newDocRef = doc(tiresCollectionRef);
                    batch.set(newDocRef, tire);
                });

                await batch.commit();
                showMessage(`Importation réussie de ${data.length} pneumatiques.`, 'success');
            } catch (error) {
                console.error("Erreur d'importation vers Firestore:", error);
                showMessage('Erreur lors de l\'importation des données. Veuillez vérifier le format du fichier.', 'error');
            } finally {
                loadingModal.style.display = 'none';
                xlsxFileInput.value = null;
                importBtn.disabled = true;
            }
        }
    </script>
</body>
</html>
