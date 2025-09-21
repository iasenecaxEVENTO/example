<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Eventalo - Descubre Eventos Locales</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.7.1/dist/leaflet.css" xintegrity="sha512-xodxW+yGgXk8nL3Jj6l6PqY2FwRz5t+4i+R1L1U4G7D6KjJmR9jX4y+D6WjJ5t6P" crossorigin=""/>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f0f2f5;
            padding-bottom: 5rem;
        }
        .scroll-container {
            overflow-x: auto;
            -webkit-overflow-scrolling: touch;
        }
        .scroll-container::-webkit-scrollbar {
            display: none;
        }
        .modal-overlay {
            background-color: rgba(0, 0, 0, 0.7);
        }
        #map {
            height: calc(100vh - 120px);
            width: 100%;
            border-radius: 1.5rem;
        }
        .leaflet-popup-content-wrapper {
            border-radius: 0.75rem;
        }
    </style>
</head>
<body class="bg-gray-100 antialiased">

    <!-- Main Container -->
    <div class="max-w-4xl mx-auto p-4">

        <!-- Header -->
        <header class="flex flex-col sm:flex-row items-center justify-between py-6">
            <h1 class="text-3xl font-bold text-gray-800 mb-4 sm:mb-0">Eventalo</h1>
            <div class="flex items-center space-x-4 w-full sm:w-auto">
                <!-- Search Bar -->
                <div class="relative w-full max-w-sm">
                    <input id="searchInput" type="text" placeholder="Buscar eventos, lugares..." class="w-full px-4 py-2 pl-10 rounded-full border border-gray-300 focus:outline-none focus:ring-2 focus:ring-indigo-500 transition-all">
                    <svg xmlns="http://www.w3.org/2000/svg" class="absolute left-3 top-1/2 -translate-y-1/2 w-5 h-5 text-gray-400" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" />
                    </svg>
                </div>
            </div>
        </header>

        <!-- Category Buttons -->
        <div class="mt-4 mb-8 scroll-container">
            <div id="categoryButtons" class="flex space-x-2 pb-2">
                <button data-category="Todos" class="category-btn bg-indigo-600 text-white font-medium px-4 py-2 rounded-full shadow-lg hover:bg-indigo-700 transition-colors whitespace-nowrap">Todos</button>
                <button data-category="Música" class="category-btn bg-white text-gray-800 font-medium px-4 py-2 rounded-full shadow-lg hover:bg-gray-200 transition-colors whitespace-nowrap">Música</button>
                <button data-category="Fiestas Locales" class="category-btn bg-white text-gray-800 font-medium px-4 py-2 rounded-full shadow-lg hover:bg-gray-200 transition-colors whitespace-nowrap">Fiestas Locales</button>
                <button data-category="Teatro" class="category-btn bg-white text-gray-800 font-medium px-4 py-2 rounded-full shadow-lg hover:bg-gray-200 transition-colors whitespace-nowrap">Teatro</button>
                <button data-category="Mercadillos" class="category-btn bg-white text-gray-800 font-medium px-4 py-2 rounded-full shadow-lg hover:bg-gray-200 transition-colors whitespace-nowrap">Mercadillos</button>
                <button data-category="Carreras Populares" class="category-btn bg-white text-gray-800 font-medium px-4 py-2 rounded-full shadow-lg hover:bg-gray-200 transition-colors whitespace-nowrap">Deporte</button>
                <button data-category="Gastronomía" class="category-btn bg-white text-gray-800 font-medium px-4 py-2 rounded-full shadow-lg hover:bg-gray-200 transition-colors whitespace-nowrap">Gastronomía</button>
            </div>
        </div>

        <!-- Event Views -->
        <div id="eventList">
            <!-- Featured Events Section -->
            <h2 class="text-xl font-bold text-gray-800 mt-6 mb-4">Destacados</h2>
            <div id="featuredEvents" class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6">
                <!-- Featured events will be rendered here -->
            </div>
            <!-- All Events Section -->
            <h2 class="text-xl font-bold text-gray-800 mt-8 mb-4">Todos los Eventos</h2>
            <div id="allEvents" class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6">
                <!-- All events will be rendered here -->
            </div>
        </div>
        
        <div id="mapContainer" class="hidden rounded-3xl overflow-hidden shadow-xl">
            <!-- Map Controls -->
            <div class="absolute top-4 left-1/2 -translate-x-1/2 z-10 w-full px-4 flex justify-center space-x-2">
                <div class="relative inline-block text-left max-w-xs">
                    <select id="provinceSelector" class="w-full px-4 py-2 rounded-full border border-gray-300 focus:outline-none focus:ring-2 focus:ring-indigo-500 transition-all cursor-pointer">
                        <option value="Valencia">Valencia</option>
                        <option value="Bizkaia">Bizkaia</option>
                    </select>
                </div>
            </div>
            <div id="mapCategoryButtons" class="mt-4 mb-8 scroll-container absolute top-20 w-full z-10 hidden">
                <div id="mapCategoryButtonsContent" class="flex space-x-2 pb-2 px-4">
                    <button data-category="Todos" class="category-btn bg-indigo-600 text-white font-medium px-4 py-2 rounded-full shadow-lg hover:bg-indigo-700 transition-colors whitespace-nowrap">Todos</button>
                    <button data-category="Música" class="category-btn bg-white text-gray-800 font-medium px-4 py-2 rounded-full shadow-lg hover:bg-gray-200 transition-colors whitespace-nowrap">Música</button>
                    <button data-category="Fiestas Locales" class="category-btn bg-white text-gray-800 font-medium px-4 py-2 rounded-full shadow-lg hover:bg-gray-200 transition-colors whitespace-nowrap">Fiestas Locales</button>
                    <button data-category="Teatro" class="category-btn bg-white text-gray-800 font-medium px-4 py-2 rounded-full shadow-lg hover:bg-gray-200 transition-colors whitespace-nowrap">Teatro</button>
                    <button data-category="Mercadillos" class="category-btn bg-white text-gray-800 font-medium px-4 py-2 rounded-full shadow-lg hover:bg-gray-200 transition-colors whitespace-nowrap">Mercadillos</button>
                    <button data-category="Carreras Populares" class="category-btn bg-white text-gray-800 font-medium px-4 py-2 rounded-full shadow-lg hover:bg-gray-200 transition-colors whitespace-nowrap">Deporte</button>
                    <button data-category="Gastronomía" class="category-btn bg-white text-gray-800 font-medium px-4 py-2 rounded-full shadow-lg hover:bg-gray-200 transition-colors whitespace-nowrap">Gastronomía</button>
                </div>
            </div>
            <div id="map"></div>
        </div>
    </div>

    <!-- Event Detail Modal -->
    <div id="eventModal" class="fixed inset-0 z-50 hidden items-center justify-center p-4">
        <div class="modal-overlay absolute inset-0"></div>
        <div class="relative bg-white w-full max-w-2xl mx-auto rounded-3xl shadow-2xl p-6 md:p-8 transform transition-transform scale-95 duration-300">
            <button id="closeModalBtn" class="absolute top-4 right-4 text-gray-500 hover:text-gray-800 transition-colors">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
                </svg>
            </button>
            <div id="modalContent">
                <!-- Modal content will be dynamically generated here -->
            </div>
        </div>
    </div>
    
    <!-- Floating Navigation Bar -->
    <nav class="fixed bottom-0 left-0 right-0 z-40 bg-white shadow-xl rounded-t-3xl p-4">
        <div class="flex justify-around">
            <button id="homeBtn" class="flex flex-col items-center text-[#f09a32] focus:outline-none" onclick="toggleView('list')">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-7 w-7" viewBox="0 0 24 24" fill="currentColor">
                    <path d="M3 5a1 1 0 011-1h16a1 1 0 110 2H4a1 1 0 01-1-1zM3 12a1 1 0 011-1h16a1 1 0 110 2H4a1 1 0 01-1-1zM3 19a1 1 0 011-1h16a1 1 0 110 2H4a1 1 0 01-1-1z" />
                </svg>
                <span class="text-xs font-medium">Lista</span>
            </button>
            <button id="mapBtn" class="flex flex-col items-center text-gray-500 focus:outline-none" onclick="toggleView('map')">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-7 w-7" viewBox="0 0 24 24" fill="currentColor">
                    <path fill-rule="evenodd" d="M12 2C8.686 2 6 4.686 6 8s2.686 6 6 6 6-2.686 6-6-2.686-6-6-6zM12 14c-1.385 0-2.5-1.115-2.5-2.5s1.115-2.5 2.5-2.5 2.5 1.115 2.5 2.5-1.115 2.5-2.5 2.5zM12 21.5c-1.385 0-2.5-1.115-2.5-2.5s1.115-2.5 2.5-2.5 2.5 1.115 2.5 2.5-1.115 2.5-2.5 2.5z" clip-rule="evenodd" />
                </svg>
                <span class="text-xs font-medium">Mapa</span>
            </button>
        </div>
    </nav>
    
    <script src="https://unpkg.com/leaflet@1.7.1/dist/leaflet.js" xintegrity="sha512-R5J6S6T6/6jQ9zC2N6E5M6bV4T6W5h/N+5E3L5x5L5Q5L5w5/gW5/wL5wL5w/g=" crossorigin=""></script>
    <script>
        const allEventsData = [
            // Eventos de Valencia
            { nombre: "Jaleo Village", fecha: "19-21 sept", hora: "11:00-21:00", localizacion: "C/ Carlos Cervera, Valencia", lat: 39.467, lng: -0.377, precio: "Gratis", categoria: "Mercadillos", descripcion: "Mercado vintage con prendas de marcas reconocidas a precios bajos.", destacado: false, provincia: "Valencia" },
            { nombre: "Billy’s Tour", fecha: "19-21 sept", hora: "11:00-21:00", localizacion: "C/ Ramón Llull, Valencia", lat: 39.479, lng: -0.347, precio: "Gratis", categoria: "Mercadillos", descripcion: "Mercado de ropa de segunda mano y vintage en Valencia.", destacado: false, provincia: "Valencia" },
            { nombre: "Patacona Fest", fecha: "20 sept", hora: "Todo el día", localizacion: "Playa de la Patacona, Alboraya", lat: 39.489, lng: -0.320, precio: "Gratis", categoria: "Música", descripcion: "Festival con música en directo, gastronomía y actividades en la playa.", destacado: true, provincia: "Valencia" },
            { nombre: "Canta Fiesta y los Divermonstruos", fecha: "20 sept", hora: "18:00", localizacion: "Teatro Olympia, Valencia", lat: 39.468, lng: -0.377, precio: "Desde 10€", categoria: "Teatro", descripcion: "Musical infantil con canciones, bailes y personajes divertidos.", destacado: false, provincia: "Valencia" },
            { nombre: "Fiestas de Beniferri", fecha: "19-21 sept", hora: "Varias actividades", localizacion: "Plaza de la Iglesia, Beniferri", lat: 39.489, lng: -0.404, precio: "Gratis", categoria: "Fiestas Locales", descripcion: "Fiestas patronales con pregón, cena popular y música con DJ.", destacado: true, provincia: "Valencia" },
            { nombre: "Moros y Cristianos del Marítimo", fecha: "20-21 sept", hora: "Tarde y noche", localizacion: "Paseo Marítimo, Valencia", lat: 39.466, lng: -0.329, precio: "Gratis", categoria: "Fiestas Locales", descripcion: "Desfiles, mercado medieval y espectáculos callejeros.", destacado: false, provincia: "Valencia" },
            { nombre: "II Cross Escolar SD Correcaminos", fecha: "4 oct", hora: "09:00", localizacion: "Parque de Marxalenes, Valencia", lat: 39.479, lng: -0.385, precio: "Gratis", categoria: "Carreras Populares", descripcion: "Carrera escolar de atletismo para jóvenes.", destacado: false, provincia: "Valencia" },
            { nombre: "Taller de paella y cata de vinos", fecha: "4 oct", hora: "12:00", localizacion: "La Albufera, Valencia", lat: 39.35, lng: -0.35, precio: "Desde 50€", categoria: "Gastronomía", descripcion: "Aprende a cocinar paella valenciana, seguida de cata de vinos.", destacado: true, provincia: "Valencia" },
            { nombre: "Ruta de la Tapa del Cabanyal", fecha: "15-20 oct", hora: "19:00", localizacion: "Barrio del Cabanyal, Valencia", lat: 39.466, lng: -0.321, precio: "Precio fijo por tapa y bebida", categoria: "Gastronomía", descripcion: "Recorrido por bares y restaurantes del barrio, con tapas especiales.", destacado: false, provincia: "Valencia" },
            
            // Eventos de Bizkaia
            { nombre: "The Colours Market", fecha: "04-05 oct", hora: "11:00-21:00", localizacion: "C/ Máximo Aguirre, Bilbao", lat: 43.2629, lng: -2.9348, precio: "Gratis", categoria: "Mercadillos", descripcion: "Mercado efímero con moda, diseño local, música y comida.", destacado: false, provincia: "Bizkaia" },
            { nombre: "Rastro 2 de Mayo", fecha: "04 oct", hora: "11:00-15:00", localizacion: "C/ Dos de Mayo, Bilbao", lat: 43.2568, lng: -2.9298, precio: "Gratis", categoria: "Mercadillos", descripcion: "Rastro alternativo de segunda mano y vintage.", destacado: false, provincia: "Bizkaia" },
            { nombre: "Ja! Bilbao", fecha: "09-20 oct", hora: "Varias", localizacion: "Sala BBK, Bilbao", lat: 43.2608, lng: -2.9329, precio: "Consultar", categoria: "Teatro", descripcion: "Festival Internacional de Literatura y Arte con Humor.", destacado: true, provincia: "Bizkaia" },
            { nombre: "Último Lunes de Gernika", fecha: "27 oct", hora: "Todo el día", localizacion: "Centro histórico de Gernika-Lumo", lat: 43.3168, lng: -2.6775, precio: "Gratis", categoria: "Fiestas Locales", descripcion: "Gran feria agrícola y gastronómica tradicional de Bizkaia.", destacado: true, provincia: "Bizkaia" },
            { nombre: "Gorlizko Herri Krosa", fecha: "05 oct", hora: "Consultar", localizacion: "Gorliz (Bizkaia)", lat: 43.407, lng: -2.934, precio: "Inscripción", categoria: "Carreras Populares", descripcion: "Carrera popular en la costa de Bizkaia.", destacado: false, provincia: "Bizkaia" },
            { nombre: "WOP Challenge", fecha: "04-05 oct", hora: "Consultar", localizacion: "Bilbao y entorno", lat: 43.263, lng: -2.935, precio: "Inscripción", categoria: "Carreras Populares", descripcion: "Reto por equipos de trail/running o trekking.", destacado: false, provincia: "Bizkaia" },
        ];

        const provinceLocations = {
            "Valencia": { lat: 39.4699, lng: -0.3774, zoom: 13 },
            "Bizkaia": { lat: 43.263, lng: -2.935, zoom: 11 }
        };

        const featuredEventsContainer = document.getElementById('featuredEvents');
        const allEventsContainer = document.getElementById('allEvents');
        const searchInput = document.getElementById('searchInput');
        const categoryButtons = document.getElementById('categoryButtons');
        const eventModal = document.getElementById('eventModal');
        const modalContent = document.getElementById('modalContent');
        const closeModalBtn = document.getElementById('closeModalBtn');
        const eventList = document.getElementById('eventList');
        const mapContainer = document.getElementById('mapContainer');
        const mapCategoryButtons = document.getElementById('mapCategoryButtons');
        const homeBtn = document.getElementById('homeBtn');
        const mapBtn = document.getElementById('mapBtn');
        const provinceSelector = document.getElementById('provinceSelector');

        let mymap = null;
        let markers = [];
        let lastClickedEvent = null;
        let currentEvents = [];
        let activeCategoryBtn = document.querySelector('#categoryButtons .category-btn.bg-indigo-600');

        function initMap() {
            const defaultLocation = provinceLocations[provinceSelector.value];
            mymap = L.map('map').setView([defaultLocation.lat, defaultLocation.lng], defaultLocation.zoom);

            L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
                attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
            }).addTo(mymap);
        }

        function renderEventCard(evento) {
            const card = document.createElement('div');
            card.className = 'bg-white p-6 rounded-3xl shadow-lg cursor-pointer hover:shadow-xl transition-shadow transform hover:scale-105';
            card.innerHTML = `
                <div class="flex items-center space-x-2 text-sm text-gray-500 mb-2">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 text-indigo-500" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z" />
                    </svg>
                    <span>${evento.fecha}</span>
                </div>
                <h3 class="text-xl font-semibold text-gray-800 mb-2">${evento.nombre}</h3>
                <p class="text-gray-600 mb-4">${evento.descripcion}</p>
                <div class="flex items-center justify-between text-sm text-gray-700">
                    <div class="flex items-center space-x-1">
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 text-green-500" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17.657 16.657L13.414 20.9a1.998 1.998 0 01-2.827 0l-4.244-4.243a8 8 0 1111.314 0z" />
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 11a3 3 0 11-6 0 3 3 0 016 0z" />
                        </svg>
                        <span>${evento.localizacion.split(',')[0]}</span>
                    </div>
                    <span class="bg-indigo-100 text-indigo-600 px-3 py-1 rounded-full text-xs font-bold">${evento.precio}</span>
                </div>
            `;
            card.addEventListener('click', () => showModal(evento));
            return card;
        }

        function showModal(evento) {
            lastClickedEvent = evento;
            modalContent.innerHTML = `
                <h2 class="text-2xl font-bold text-gray-900 mb-2">${evento.nombre}</h2>
                <div class="flex items-center space-x-2 text-md text-gray-600 mb-4">
                    <span class="bg-gray-200 text-gray-700 font-semibold px-2 py-1 rounded-md">${evento.categoria}</span>
                    <span class="bg-gray-200 text-gray-700 font-semibold px-2 py-1 rounded-md">${evento.precio}</span>
                </div>
                <div class="space-y-4 text-gray-700">
                    <p class="flex items-center">
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-2 text-indigo-500" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z" />
                        </svg>
                        <span class="font-semibold">Fecha:</span> ${evento.fecha}
                    </p>
                    <p class="flex items-center">
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-2 text-indigo-500" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z" />
                        </svg>
                        <span class="font-semibold">Hora:</span> ${evento.hora}
                    </p>
                    <p class="flex items-center">
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-2 text-indigo-500" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17.657 16.657L13.414 20.9a1.998 1.998 0 01-2.827 0l-4.244-4.243a8 8 0 1111.314 0z" />
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 11a3 3 0 11-6 0 3 3 0 016 0z" />
                        </svg>
                        <span class="font-semibold">Lugar:</span> ${evento.localizacion}
                    </p>
                    <p class="text-lg leading-relaxed">${evento.descripcion}</p>
                </div>
            `;
            eventModal.style.display = 'flex';
        }

        function hideModal() {
            eventModal.style.display = 'none';
        }

        function filterAndRenderEvents() {
            const searchTerm = searchInput.value.toLowerCase();
            const selectedCategory = activeCategoryBtn.dataset.category;

            const filteredEvents = currentEvents.filter(evento => {
                const matchesSearch = evento.nombre.toLowerCase().includes(searchTerm) || evento.descripcion.toLowerCase().includes(searchTerm);
                const matchesCategory = selectedCategory === 'Todos' || evento.categoria === selectedCategory;
                return matchesSearch && matchesCategory;
            });

            featuredEventsContainer.innerHTML = '';
            allEventsContainer.innerHTML = '';

            filteredEvents.forEach(evento => {
                if (evento.destacado) {
                    featuredEventsContainer.appendChild(renderEventCard(evento));
                } else {
                    allEventsContainer.appendChild(renderEventCard(evento));
                }
            });
        }
        
        function updateMapMarkers() {
            markers.forEach(marker => mymap.removeLayer(marker));
            markers = [];

            const searchTerm = searchInput.value.toLowerCase();
            const selectedCategory = activeCategoryBtn.dataset.category;

            const filteredEvents = currentEvents.filter(evento => {
                const matchesSearch = evento.nombre.toLowerCase().includes(searchTerm) || evento.descripcion.toLowerCase().includes(searchTerm);
                const matchesCategory = selectedCategory === 'Todos' || evento.categoria === selectedCategory;
                return matchesSearch && matchesCategory;
            });

            filteredEvents.forEach(evento => {
                if (evento.lat && evento.lng) {
                    const marker = L.marker([evento.lat, evento.lng]).addTo(mymap);
                    marker.bindPopup(`
                        <strong class="text-indigo-600">${evento.nombre}</strong><br>
                        ${evento.localizacion.split(',')[0]}<br>
                        <button onclick="showModal(${JSON.stringify(evento).split('"').join("&#34;")})" class="mt-2 text-indigo-500 hover:underline">Ver detalles</button>
                    `);
                    markers.push(marker);
                }
            });
        }

        function filterByProvince(province) {
            currentEvents = allEventsData.filter(evento => evento.provincia === province);
            
            if (eventList.classList.contains('hidden')) {
                const location = provinceLocations[province];
                mymap.setView([location.lat, location.lng], location.zoom);
                updateMapMarkers();
            } else {
                filterAndRenderEvents();
            }
        }

        function toggleView(view) {
            if (view === 'list') {
                eventList.classList.remove('hidden');
                mapContainer.classList.add('hidden');
                mapCategoryButtons.classList.add('hidden');
                homeBtn.classList.add('text-[#f09a32]');
                homeBtn.classList.remove('text-gray-500');
                mapBtn.classList.add('text-gray-500');
                mapBtn.classList.remove('text-[#f09a32]');
                
                // Sincronizar el botón activo de la lista
                document.querySelectorAll('#categoryButtons .category-btn').forEach(btn => {
                    btn.classList.remove('bg-indigo-600', 'text-white');
                    btn.classList.add('bg-white', 'text-gray-800');
                });
                activeCategoryBtn.classList.remove('bg-white', 'text-gray-800');
                activeCategoryBtn.classList.add('bg-indigo-600', 'text-white');

                filterAndRenderEvents();
            } else if (view === 'map') {
                eventList.classList.add('hidden');
                mapContainer.classList.remove('hidden');
                mapCategoryButtons.classList.remove('hidden');
                homeBtn.classList.remove('text-[#f09a32]');
                homeBtn.classList.add('text-gray-500');
                mapBtn.classList.remove('text-gray-500');
                mapBtn.classList.add('text-[#f09a32]');

                setTimeout(() => {
                    mymap.invalidateSize();
                }, 200);

                if (lastClickedEvent && lastClickedEvent.lat && lastClickedEvent.lng) {
                    mymap.setView([lastClickedEvent.lat, lastClickedEvent.lng], 15);
                } else {
                    const location = provinceLocations[provinceSelector.value];
                    mymap.setView([location.lat, location.lng], location.zoom);
                }
                
                updateMapMarkers();
            }
        }

        // Event Listeners
        searchInput.addEventListener('input', () => {
            if (eventList.classList.contains('hidden')) {
                updateMapMarkers();
            } else {
                filterAndRenderEvents();
            }
        });
        closeModalBtn.addEventListener('click', hideModal);
        eventModal.addEventListener('click', (e) => {
            if (e.target.classList.contains('modal-overlay')) {
                hideModal();
            }
        });

        function handleCategoryClick(e) {
            if (e.target.matches('.category-btn')) {
                const category = e.target.dataset.category;
                activeCategoryBtn = document.querySelector(`.category-btn[data-category="${category}"]`);

                document.querySelectorAll('.category-btn').forEach(btn => {
                    btn.classList.remove('bg-indigo-600', 'text-white');
                    btn.classList.add('bg-white', 'text-gray-800');
                });
                
                document.querySelectorAll(`.category-btn[data-category="${category}"]`).forEach(btn => {
                    btn.classList.remove('bg-white', 'text-gray-800');
                    btn.classList.add('bg-indigo-600', 'text-white');
                });

                lastClickedEvent = null;
                
                if (eventList.classList.contains('hidden')) {
                    updateMapMarkers();
                } else {
                    filterAndRenderEvents();
                }
            }
        }

        categoryButtons.addEventListener('click', handleCategoryClick);
        document.getElementById('mapCategoryButtonsContent').addEventListener('click', handleCategoryClick);

        provinceSelector.addEventListener('change', (e) => {
            lastClickedEvent = null;
            filterByProvince(e.target.value);
        });

        window.onload = () => {
            initMap();
            filterByProvince(provinceSelector.value);
        };
    </script>
</body>
</html>
# example
