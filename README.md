<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cooperative Directory</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
    <!-- PapaParse CDN for parsing CSV -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.4.1/papaparse.min.js"></script>
</head>
<body class="bg-gray-50 font-sans text-gray-800">

    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
        <!-- Header -->
        <div class="mb-8 text-center sm:text-left sm:flex sm:items-center sm:justify-between border-b border-gray-200 pb-6">
            <div>
                <h1 class="text-3xl font-bold text-gray-900 tracking-tight">Region VI Cooperative Directory</h1>
                <p class="mt-2 text-sm text-gray-600">Search by name or address and filter activities. Click any row to view contact details.</p>
            </div>
            <!-- Sync Status Indicator -->
            <div id="syncStatus" class="mt-4 sm:mt-0 text-xs font-semibold px-3 py-1.5 rounded-full bg-yellow-100 text-yellow-800 animate-pulse">
                Connecting to Data Source...
            </div>
        </div>

        <!-- Filters Section -->
        <div class="grid grid-cols-1 gap-4 md:grid-cols-3 mb-6 bg-white p-4 rounded-lg shadow-sm border border-gray-100">
            <div>
                <label for="nameSearch" class="block text-sm font-medium text-gray-700 mb-1">Search by Cooperative Name</label>
                <input type="text" id="nameSearch" placeholder="Type cooperative name..." 
                       class="block w-full rounded-md border-gray-300 focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm p-2.5 border">
            </div>
            <div>
                <label for="addressSearch" class="block text-sm font-medium text-gray-700 mb-1">Search by Address</label>
                <input type="text" id="addressSearch" placeholder="Type address or location..." 
                       class="block w-full rounded-md border-gray-300 focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm p-2.5 border">
            </div>
            <div>
                <label for="businessFilter" class="block text-sm font-medium text-gray-700 mb-1">Filter by Business Type</label>
                <select id="businessFilter" 
                        class="block w-full rounded-md border-gray-300 focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm p-2.5 border bg-white">
                    <option value="">All Businesses</option>
                </select>
            </div>
        </div>

        <!-- Error/Fallback Banner (Hidden by default) -->
        <div id="statusMessage" class="hidden mb-4 p-3 rounded-md text-sm"></div>

        <!-- Table Container -->
        <div class="bg-white shadow-sm rounded-lg border border-gray-200 overflow-hidden">
            <div class="overflow-x-auto">
                <table class="min-w-full divide-y divide-gray-200">
                    <thead class="bg-gray-50">
                        <tr>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-semibold text-gray-500 uppercase tracking-wider">Cooperative Name</th>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-semibold text-gray-500 uppercase tracking-wider">Address</th>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-semibold text-gray-500 uppercase tracking-wider">Business Activities</th>
                        </tr>
                    </thead>
                    <tbody id="tableBody" class="bg-white divide-y divide-gray-200">
                        <tr>
                            <td colspan="3" class="px-6 py-10 text-center text-sm text-gray-500">Loading directory entries...</td>
                        </tr>
                    </tbody>
                </table>
            </div>
            <!-- Empty state -->
            <div id="emptyState" class="hidden text-center py-12 bg-white">
                <svg class="mx-auto h-12 w-12 text-gray-400" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9.172 16.172a4 4 0 015.656 0M9 10h.01M15 10h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
                </svg>
                <h3 class="mt-2 text-sm font-medium text-gray-900">No results found</h3>
                <p class="mt-1 text-sm text-gray-500">Try adjusting your search filters.</p>
            </div>
        </div>
    </div>

    <!-- Cooperative Detail Card Modal Overlay -->
    <div id="detailModal" class="fixed inset-0 bg-black/50 z-50 flex items-center justify-center p-4 hidden opacity-0 transition-opacity duration-300">
        <div class="bg-white rounded-xl shadow-2xl max-w-lg w-full overflow-hidden transform scale-95 transition-transform duration-300 border border-gray-100">
            <!-- Modal Header Accent -->
            <div class="h-2 bg-gradient-to-r from-indigo-500 to-purple-600"></div>
            
            <div class="p-6">
                <!-- Title & Close -->
                <div class="flex justify-between items-start mb-4">
                    <h2 id="modalCoopName" class="text-2xl font-bold text-gray-900 tracking-tight leading-tight">Cooperative Details</h2>
                    <button id="closeModalBtn" class="text-gray-400 hover:text-gray-600 p-1 rounded-lg hover:bg-gray-100 transition-colors">
                        <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path></svg>
                    </button>
                </div>

                <!-- Card Content -->
                <div class="space-y-4">
                    <div>
                        <span class="block text-xs font-semibold text-gray-400 uppercase tracking-wider mb-0.5">Location Address</span>
                        <p id="modalAddress" class="text-gray-700 text-sm bg-gray-50 p-2.5 rounded-lg border border-gray-100 font-medium"></p>
                    </div>

                    <!-- Contact details ONLY visible here inside the card -->
                    <div class="grid grid-cols-2 gap-4">
                        <div>
                            <span class="block text-xs font-semibold text-gray-400 uppercase tracking-wider mb-0.5">Contact Number</span>
                            <p id="modalContact" class="text-gray-700 text-sm font-semibold bg-gray-50 p-2.5 rounded-lg border border-gray-100"></p>
                        </div>
                        <div>
                            <span class="block text-xs font-semibold text-gray-400 uppercase tracking-wider mb-0.5">Email Address</span>
                            <div id="modalEmailContainer" class="bg-gray-50 p-2.5 rounded-lg border border-gray-100 overflow-hidden text-ellipsis">
                                <!-- Populated dynamically -->
                            </div>
                        </div>
                    </div>

                    <div>
                        <span class="block text-xs font-semibold text-gray-400 uppercase tracking-wider mb-1.5">Business Portfolio</span>
                        <div id="modalBusinesses" class="flex flex-wrap gap-1.5">
                            <!-- Populated dynamically -->
                        </div>
                    </div>
                </div>
            </div>
            
            <!-- Modal Footer -->
            <div class="bg-gray-50 px-6 py-3 border-t border-gray-100 flex justify-end">
                <button id="closeModalBottomBtn" class="px-4 py-2 bg-indigo-600 hover:bg-indigo-700 text-white font-medium text-sm rounded-lg transition-colors shadow-sm">
                    Close Details
                </button>
            </div>
        </div>
    </div>

    <script>
        // Set your live Google Sheet CSV URL here, or leave empty to fetch local file "CoopBiz.csv"
        const googleSheetUrl = "https://docs.google.com/spreadsheets/d/e/2PACX-1vSFDmfxvYSeZccXSuNVyKbAU1A6VOxrCmQVwu3F1FpafcC6jJ5hqCeKVwy1yuGI3zhysVhJcwEgKQdM/pub?output=csv"; 

        // Local fallback dataset
        const embeddedFallback = `Cooperative Name,Address,Contact Number,Email,Business
ABC Cooperative,Leganes LCC,912345678,abc_coop@gmail.com,"Agri Machineries, Palay Trading, Mushroom"
DEF Cooperative,Pavia PCC,912345677,def_coop@gmail.com,"Coconut Crafts, Coco Balls, Coconut Husk"
GHI Cooperative,Oton OCC,912345777,GHI_coop@gmail.com,"Coconut Crafts, Agrimachineries, Corn"`;

        let csvDataArray = [];

        // DOM elements
        const nameSearchInput = document.getElementById('nameSearch');
        const addressSearchInput = document.getElementById('addressSearch');
        const businessFilterSelect = document.getElementById('businessFilter');
        const tableBody = document.getElementById('tableBody');
        const emptyState = document.getElementById('emptyState');
        const syncStatus = document.getElementById('syncStatus');
        const statusMessage = document.getElementById('statusMessage');

        // Modal Elements
        const detailModal = document.getElementById('detailModal');
        const modalContainer = detailModal.querySelector('div');
        const modalCoopName = document.getElementById('modalCoopName');
        const modalAddress = document.getElementById('modalAddress');
        const modalContact = document.getElementById('modalContact');
        const modalEmailContainer = document.getElementById('modalEmailContainer');
        const modalBusinesses = document.getElementById('modalBusinesses');

        window.addEventListener('DOMContentLoaded', () => {
            const targetUrl = googleSheetUrl && !googleSheetUrl.includes("YOUR_GOOGLE_SHEET") ? googleSheetUrl : 'CoopBiz.csv';
            
            fetch(targetUrl)
                .then(response => {
                    if (!response.ok) throw new Error('Data endpoint unreachable');
                    return response.text();
                })
                .then(csvText => {
                    updateSyncBadge(targetUrl === 'CoopBiz.csv' ? '● File Mode' : '● Connected Live', 'bg-green-100 text-green-800');
                    parseAndProcessCSV(csvText);
                })
                .catch(error => {
                    console.warn('Network path failed, dropping back to inline preview dataset:', error);
                    showStatus('Data source unreachable via network fetch (local file direct view mode). Showing fallback dataset.', 'bg-blue-50 text-blue-800');
                    updateSyncBadge('Preview Mode', 'bg-blue-100 text-blue-800');
                    parseAndProcessCSV(embeddedFallback);
                });

            initEventListeners();
        });

        function initEventListeners() {
            nameSearchInput.addEventListener('input', applyFilters);
            addressSearchInput.addEventListener('input', applyFilters);
            businessFilterSelect.addEventListener('change', applyFilters);

            // Modal closing bindings
            document.getElementById('closeModalBtn').addEventListener('click', closeModal);
            document.getElementById('closeModalBottomBtn').addEventListener('click', closeModal);
            detailModal.addEventListener('click', (e) => { if (e.target === detailModal) closeModal(); });
            window.addEventListener('keydown', (e) => { if (e.key === 'Escape') closeModal(); });
        }

        function updateSyncBadge(text, classes) {
            syncStatus.className = `mt-4 sm:mt-0 text-xs font-semibold px-3 py-1.5 rounded-full ${classes}`;
            syncStatus.textContent = text;
        }

        function showStatus(msg, classes) {
            statusMessage.className = `mb-4 p-3 rounded-md text-sm ${classes}`;
            statusMessage.textContent = msg;
            statusMessage.classList.remove('hidden');
        }

        function parseAndProcessCSV(csvText) {
            Papa.parse(csvText, {
                header: true,
                skipEmptyLines: true,
                dynamicTyping: true,
                complete: function(results) {
                    csvDataArray = results.data;
                    populateBusinessDropdown();
                    applyFilters();
                }
            });
        }

        function populateBusinessDropdown() {
            const businessSet = new Set();
            csvDataArray.forEach(row => {
                if (row['Business']) {
                    row['Business'].split(',').forEach(b => {
                        const trimmed = b.trim();
                        if (trimmed) businessSet.add(trimmed);
                    });
                }
            });

            const sortedBusinesses = Array.from(businessSet).sort();
            businessFilterSelect.innerHTML = '<option value="">All Businesses</option>';
            sortedBusinesses.forEach(biz => {
                const option = document.createElement('option');
                option.value = biz;
                option.textContent = biz;
                businessFilterSelect.appendChild(option);
            });
        }

        function applyFilters() {
            const searchName = nameSearchInput.value.toLowerCase().trim();
            const searchAddress = addressSearchInput.value.toLowerCase().trim();
            const selectedBusiness = businessFilterSelect.value;

            const filteredData = csvDataArray.filter(row => {
                const rowName = String(row['Cooperative Name'] || '').toLowerCase();
                const rowAddress = String(row['Address'] || '').toLowerCase();
                
                const matchesName = rowName.includes(searchName);
                const matchesAddress = rowAddress.includes(searchAddress);

                let matchesBusiness = true;
                if (selectedBusiness && row['Business']) {
                    const items = row['Business'].split(',').map(item => item.trim());
                    matchesBusiness = items.includes(selectedBusiness);
                } else if (selectedBusiness) {
                    matchesBusiness = false;
                }

                return matchesName && matchesAddress && matchesBusiness;
            });

            renderTable(filteredData);
        }

        function renderTable(data) {
            tableBody.innerHTML = '';
            if (data.length === 0) {
                emptyState.classList.remove('hidden');
                return;
            } else {
                emptyState.classList.add('hidden');
            }

            data.forEach((row, index) => {
                const tr = document.createElement('tr');
                tr.className = 'hover:bg-indigo-50/50 cursor-pointer transition-colors duration-150 group';
                tr.dataset.index = index; 

                let businessBadgesHtml = '';
                if (row['Business']) {
                    row['Business'].split(',').forEach(b => {
                        const trimmed = b.trim();
                        if (trimmed) {
                            businessBadgesHtml += `<span class="inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium bg-indigo-100 text-indigo-800 mr-1.5 mb-1">${trimmed}</span>`;
                        }
                    });
                }

                // Table now strictly renders Name, Address, and Business badges
                tr.innerHTML = `
                    <td class="px-6 py-4 whitespace-nowrap text-sm font-semibold text-gray-900 group-hover:text-indigo-600 transition-colors">${row['Cooperative Name'] || '-'}</td>
                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-600">${row['Address'] || '-'}</td>
                    <td class="px-6 py-4 text-sm text-gray-600 max-w-xs sm:max-w-md">
                        <div class="flex flex-wrap">${businessBadgesHtml || '-'}</div>
                    </td>
                `;

                tr.addEventListener('click', () => {
                    openDetailModal(row);
                });

                tableBody.appendChild(tr);
            });
        }

        function openDetailModal(data) {
            modalCoopName.textContent = data['Cooperative Name'] || 'No Name Provided';
            modalAddress.textContent = data['Address'] || 'No Address Listed';
            modalContact.textContent = data['Contact Number'] || 'None';
            
            if (data['Email']) {
                modalEmailContainer.innerHTML = `<a href="mailto:${data['Email']}" class="text-indigo-600 hover:text-indigo-900 font-semibold underline text-sm break-all">${data['Email']}</a>`;
            } else {
                modalEmailContainer.innerHTML = `<span class="text-gray-400 text-sm">None</span>`;
            }

            modalBusinesses.innerHTML = '';
            if (data['Business']) {
                data['Business'].split(',').forEach(b => {
                    const trimmed = b.trim();
                    if (trimmed) {
                        const span = document.createElement('span');
                        span.className = "inline-flex items-center px-3 py-1 rounded-full text-xs font-semibold bg-gradient-to-r from-indigo-50 to-purple-50 text-indigo-700 border border-indigo-100 shadow-sm";
                        span.textContent = trimmed;
                        modalBusinesses.appendChild(span);
                    }
                });
            } else {
                modalBusinesses.innerHTML = `<span class="text-gray-400 text-sm">No business segments declared</span>`;
            }

            detailModal.classList.remove('hidden');
            setTimeout(() => {
                detailModal.classList.remove('opacity-0');
                modalContainer.classList.remove('scale-95');
            }, 10);
        }

        function closeModal() {
            detailModal.classList.add('opacity-0');
            modalContainer.classList.add('scale-95');
            setTimeout(() => {
                detailModal.classList.add('hidden');
            }, 300);
        }
    </script>
</body>
</html>
