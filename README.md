<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>歌曲搜尋平台</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@400;500;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Noto Sans TC', sans-serif;
            background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
            min-height: 100vh;
        }
        .song-card {
            transition: all 0.3s ease;
        }
        .song-card:hover {
            transform: translateY(-5px);
            box-shadow: 0 10px 20px rgba(0,0,0,0.1);
        }
        .search-input:focus {
            box-shadow: 0 0 0 3px rgba(99, 102, 241, 0.3);
        }
        .filter-btn.active {
            background-color: #4f46e5;
            color: white;
        }
        .category-btn {
            transition: all 0.2s ease;
        }
        .category-btn.active {
            background-color: #8b5cf6;
            color: white;
            transform: translateY(-2px);
            box-shadow: 0 4px 6px rgba(139, 92, 246, 0.2);
        }
        .loader {
            border-top-color: #4f46e5;
            animation: spinner 0.6s linear infinite;
        }
        @keyframes spinner {
            to {transform: rotate(360deg);}
        }
        .language-tag {
            white-space: nowrap;
        }
        .category-tag {
            display: inline-block;
            margin-right: 0.25rem;
            margin-bottom: 0.25rem;
            cursor: pointer;
        }
        .active-filters {
            transition: all 0.3s ease;
            max-height: 0;
            overflow: hidden;
        }
        .active-filters.show {
            max-height: 100px;
            margin-bottom: 1rem;
        }
        .copy-btn {
            transition: all 0.2s ease;
        }
        .copy-btn:hover {
            transform: translateY(-2px);
            animation: bounce 0.5s ease;
        }
        @keyframes bounce {
            0%, 100% { transform: translateY(-2px); }
            50% { transform: translateY(-6px); }
        }
        .copy-tooltip {
            position: absolute;
            top: -30px;
            right: 0;
            background-color: #4f46e5;
            color: white;
            padding: 4px 8px;
            border-radius: 4px;
            font-size: 12px;
            opacity: 0;
            transition: opacity 0.3s ease;
            pointer-events: none;
        }
        .copy-tooltip.show {
            opacity: 1;
        }
        .copy-tooltip:after {
            content: '';
            position: absolute;
            top: 100%;
            right: 10px;
            border-width: 5px;
            border-style: solid;
            border-color: #4f46e5 transparent transparent transparent;
        }
        .category-count {
            display: inline-flex;
            align-items: center;
            justify-content: center;
            min-width: 20px;
            height: 20px;
            padding: 0 6px;
            border-radius: 10px;
            background-color: rgba(139, 92, 246, 0.2);
            color: #6d28d9;
            font-size: 12px;
            font-weight: 600;
            margin-left: 6px;
        }
        .category-btn.active .category-count {
            background-color: rgba(255, 255, 255, 0.3);
            color: white;
        }
        .filter-tag {
            cursor: pointer;
            font-size: 0.75rem;
            padding: 0.25rem 0.5rem;
            border-radius: 9999px;
            display: inline-flex;
            align-items: center;
            margin-right: 0.5rem;
            margin-bottom: 0.25rem;
        }
    </style>
</head>
<body>
    <div class="container mx-auto px-4 py-8">
        <header class="text-center mb-10">
            <h1 class="text-4xl font-bold text-indigo-800 mb-2">我的歌曲庫</h1>
            <p class="text-gray-600 text-lg">搜尋、瀏覽我會唱的歌曲</p>
        </header>

        <div class="bg-white rounded-xl shadow-lg p-6 mb-8">
            <!-- Search bar - moved to the top -->
            <div class="mb-6">
                <div class="relative">
                    <input 
                        type="text" 
                        id="search" 
                        placeholder="搜尋歌曲、歌手或語言..." 
                        class="search-input w-full px-4 py-3 rounded-lg border border-gray-300 focus:outline-none text-gray-700"
                    >
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 absolute right-3 top-3.5 text-gray-400" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" />
                    </svg>
                </div>
            </div>

            <!-- Category filters -->
            <div class="mb-6">
                <div class="flex justify-between items-center mb-3">
                    <h3 class="text-lg font-semibold text-gray-700">分類</h3>
                </div>
                <div class="flex gap-2 flex-wrap" id="category-filters">
                    <!-- Category filters will be dynamically added here -->
                </div>
                
                <!-- Active filters display with clear button moved to the left -->
                <div class="flex items-center mt-3">
                    <button id="clear-categories" class="text-sm text-purple-600 hover:text-purple-800 font-medium mr-2 px-3 py-1 bg-purple-50 rounded-lg hover:bg-purple-100 transition-colors">清除所有</button>
                    <div id="active-filters" class="active-filters flex-grow px-3 py-2 bg-purple-50 rounded-lg">
                        <div class="flex flex-wrap gap-1" id="active-filters-list">
                            <!-- Active filters will be shown here -->
                        </div>
                    </div>
                </div>
            </div>

            <div class="flex flex-col md:flex-row md:items-center gap-4 mb-6">
                <div class="flex gap-2 flex-wrap" id="language-filters">
                    <!-- Language filters will be dynamically added here -->
                </div>
            </div>

            <div class="text-gray-600 mb-4">
                <span id="result-count" class="font-medium">0</span> 首歌曲
            </div>

            <div id="loading" class="flex justify-center py-10">
                <div class="loader h-10 w-10 rounded-full border-4 border-gray-200"></div>
            </div>

            <div id="songs-container" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4 hidden">
                <!-- Songs will be populated here -->
            </div>

            <div id="no-results" class="text-center py-10 hidden">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-16 w-16 mx-auto text-gray-400 mb-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9.172 16.172a4 4 0 015.656 0M9 10h.01M15 10h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
                </svg>
                <p class="text-gray-600 text-lg">沒有找到符合的歌曲</p>
            </div>
        </div>

        <div class="bg-white rounded-xl shadow-lg p-6">
            <h2 class="text-2xl font-bold text-indigo-800 mb-4">統計資訊</h2>
            <div id="stats-container" class="grid grid-cols-2 md:grid-cols-4 gap-4">
                <div class="bg-indigo-50 rounded-lg p-4 text-center">
                    <div class="text-3xl font-bold text-indigo-600" id="total-count">0</div>
                    <div class="text-gray-600">總歌曲數</div>
                </div>
                <!-- Additional stats will be added dynamically -->
            </div>
        </div>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', function() {
            const songsContainer = document.getElementById('songs-container');
            const searchInput = document.getElementById('search');
            const resultCount = document.getElementById('result-count');
            const noResults = document.getElementById('no-results');
            const loading = document.getElementById('loading');
            const languageFilters = document.getElementById('language-filters');
            const categoryFilters = document.getElementById('category-filters');
            const activeFilters = document.getElementById('active-filters');
            const activeFiltersList = document.getElementById('active-filters-list');
            const clearCategoriesBtn = document.getElementById('clear-categories');
            const statsContainer = document.getElementById('stats-container');
            const totalCountEl = document.getElementById('total-count');
            
            let allSongs = [];
            let currentLanguageFilter = 'all';
            let selectedCategories = new Set(); // Store selected categories
            let languages = new Set();
            let categories = new Set();
            let categoryCounts = {}; // Store category counts

            // Google Sheet ID from the provided URL
            const sheetId = '1z-kl4dUioRrhJaq82O283991VPtg2PfXqx4eN5fQ9-8';
            
            // Use the opensheet.elk.sh API to fetch the data
            fetchSheetData();

            function fetchSheetData() {
                const apiUrl = `https://opensheet.elk.sh/${sheetId}/sheet1`;
                
                fetch(apiUrl)
                    .then(response => {
                        if (!response.ok) {
                            throw new Error('Network response was not ok');
                        }
                        return response.json();
                    })
                    .then(data => {
                        processSheetData(data);
                    })
                    .catch(error => {
                        console.error('Error fetching sheet data:', error);
                        showFetchError();
                    });
            }

            function processSheetData(data) {
                try {
                    // Map the data to our songs array
                    allSongs = data.map((row, index) => {
                        const song = {
                            id: index + 1,
                            title: row.歌名 || row.title || row.Song || '',
                            artist: row.歌手 || row.artist || row.Artist || '',
                            language: row.語言 || row.language || row.Language || '',
                            category: row.分類 || row.category || row.Category || '',
                            notes: row.備註 || row.notes || row.Notes || ''
                        };
                        
                        // Add language to our set of languages
                        if (song.language) {
                            languages.add(song.language);
                        }
                        
                        // Process categories (split by comma)
                        if (song.category) {
                            const categoryList = song.category.split(',').map(cat => cat.trim());
                            song.categories = categoryList; // Store as array for filtering
                            
                            // Add each category to our set and count
                            categoryList.forEach(cat => {
                                if (cat) {
                                    categories.add(cat);
                                    categoryCounts[cat] = (categoryCounts[cat] || 0) + 1;
                                }
                            });
                        } else {
                            song.categories = [];
                        }
                        
                        return song;
                    });
                    
                    // Create language filters (without "All" button)
                    createLanguageFilters();
                    
                    // Create category filters
                    createCategoryFilters();
                    
                    // Update statistics
                    updateStats();
                    
                    // Initial render
                    renderSongs(allSongs);
                } catch (error) {
                    console.error('Error processing sheet data:', error);
                    showFetchError();
                } finally {
                    loading.classList.add('hidden');
                    songsContainer.classList.remove('hidden');
                }
            }

            function showFetchError() {
                loading.classList.add('hidden');
                songsContainer.classList.add('hidden');
                noResults.classList.remove('hidden');
                noResults.innerHTML = `
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-16 w-16 mx-auto text-red-400 mb-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z" />
                    </svg>
                    <p class="text-gray-600 text-lg">無法獲取歌曲數據</p>
                    <p class="text-gray-500 mt-2">請確保Google Sheet格式正確，並且可以公開訪問</p>
                    <button id="retry-btn" class="mt-4 px-4 py-2 bg-indigo-600 text-white rounded-lg hover:bg-indigo-700">重試</button>
                `;
                
                // Add retry functionality
                document.getElementById('retry-btn').addEventListener('click', fetchSheetData);
            }

            function createLanguageFilters() {
                // Clear existing language filters
                languageFilters.innerHTML = '';
                
                // Add a filter button for each language
                Array.from(languages).sort().forEach(language => {
                    const button = document.createElement('button');
                    button.className = 'filter-btn px-4 py-2 rounded-lg bg-gray-100 hover:bg-gray-200 text-gray-700 font-medium';
                    button.dataset.filter = language;
                    button.textContent = language;
                    
                    button.addEventListener('click', function() {
                        document.querySelectorAll('.filter-btn').forEach(btn => btn.classList.remove('active'));
                        this.classList.add('active');
                        currentLanguageFilter = this.dataset.filter;
                        filterSongs();
                    });
                    
                    languageFilters.appendChild(button);
                });
            }

            function createCategoryFilters() {
                // Clear existing category filters
                categoryFilters.innerHTML = '';
                
                // Sort categories by count (highest to lowest)
                const sortedCategories = Array.from(categories).sort((a, b) => {
                    return (categoryCounts[b] || 0) - (categoryCounts[a] || 0);
                });
                
                // Add a filter button for each category
                sortedCategories.forEach(category => {
                    const button = document.createElement('button');
                    button.className = 'category-btn px-4 py-2 rounded-lg bg-purple-100 hover:bg-purple-200 text-purple-700 font-medium flex items-center';
                    button.dataset.category = category;
                    
                    // Add category name and count
                    const count = categoryCounts[category] || 0;
                    button.innerHTML = `
                        <span>${category}</span>
                        <span class="category-count">${count}</span>
                    `;
                    
                    button.addEventListener('click', function() {
                        // Toggle active state
                        this.classList.toggle('active');
                        
                        // Update selected categories set
                        const category = this.dataset.category;
                        if (selectedCategories.has(category)) {
                            selectedCategories.delete(category);
                        } else {
                            selectedCategories.add(category);
                        }
                        
                        // Update active filters display
                        updateActiveFilters();
                        
                        // Filter songs
                        filterSongs();
                    });
                    
                    categoryFilters.appendChild(button);
                });
                
                // Clear categories button
                clearCategoriesBtn.addEventListener('click', function() {
                    selectedCategories.clear();
                    document.querySelectorAll('.category-btn').forEach(btn => btn.classList.remove('active'));
                    updateActiveFilters();
                    filterSongs();
                });
            }

            function updateActiveFilters() {
                activeFiltersList.innerHTML = '';
                
                if (selectedCategories.size === 0) {
                    activeFilters.classList.remove('show');
                    return;
                }
                
                activeFilters.classList.add('show');
                
                // Add a chip for each selected category
                selectedCategories.forEach(category => {
                    const chip = document.createElement('div');
                    chip.className = 'filter-tag bg-purple-200 text-purple-800';
                    chip.dataset.category = category;
                    chip.innerHTML = `
                        <span>${category}</span>
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-3 w-3 ml-1" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
                        </svg>
                    `;
                    
                    // Add click functionality to the entire tag
                    chip.addEventListener('click', function() {
                        const category = this.dataset.category;
                        selectedCategories.delete(category);
                        
                        // Update category button state
                        const categoryBtn = document.querySelector(`.category-btn[data-category="${category}"]`);
                        if (categoryBtn) {
                            categoryBtn.classList.remove('active');
                        }
                        
                        updateActiveFilters();
                        filterSongs();
                    });
                    
                    activeFiltersList.appendChild(chip);
                });
            }

            // Search functionality
            searchInput.addEventListener('input', function() {
                filterSongs();
            });

            function filterSongs() {
                const searchTerm = searchInput.value.toLowerCase();
                let filteredSongs = allSongs;
                
                // Apply language filter
                if (currentLanguageFilter !== 'all') {
                    filteredSongs = filteredSongs.filter(song => song.language === currentLanguageFilter);
                }
                
                // Apply category filters (intersection - song must have ALL selected categories)
                if (selectedCategories.size > 0) {
                    filteredSongs = filteredSongs.filter(song => {
                        // Check if song has all selected categories
                        for (const category of selectedCategories) {
                            if (!song.categories || !song.categories.includes(category)) {
                                return false;
                            }
                        }
                        return true;
                    });
                }
                
                // Apply search filter
                if (searchTerm) {
                    filteredSongs = filteredSongs.filter(song => 
                        (song.title && song.title.toLowerCase().includes(searchTerm)) || 
                        (song.artist && song.artist.toLowerCase().includes(searchTerm)) ||
                        (song.language && song.language.toLowerCase().includes(searchTerm)) ||
                        (song.category && song.category.toLowerCase().includes(searchTerm)) ||
                        (song.notes && song.notes.toLowerCase().includes(searchTerm))
                    );
                }
                
                // Update category counts based on filtered songs
                updateCategoryCounts(filteredSongs);
                
                renderSongs(filteredSongs);
            }

            function updateCategoryCounts(filteredSongs) {
                // Reset counts
                const filteredCategoryCounts = {};
                
                // Count categories in filtered songs
                filteredSongs.forEach(song => {
                    if (song.categories && song.categories.length > 0) {
                        song.categories.forEach(cat => {
                            if (cat) {
                                filteredCategoryCounts[cat] = (filteredCategoryCounts[cat] || 0) + 1;
                            }
                        });
                    }
                });
                
                // Update category buttons with new counts
                document.querySelectorAll('.category-btn').forEach(btn => {
                    const category = btn.dataset.category;
                    const countElement = btn.querySelector('.category-count');
                    if (countElement) {
                        countElement.textContent = filteredCategoryCounts[category] || 0;
                    }
                });
            }

            function renderSongs(songs) {
                songsContainer.innerHTML = '';
                resultCount.textContent = songs.length;
                
                if (songs.length === 0) {
                    noResults.classList.remove('hidden');
                    songsContainer.classList.add('hidden');
                } else {
                    noResults.classList.add('hidden');
                    songsContainer.classList.remove('hidden');
                    
                    songs.forEach(song => {
                        const songCard = document.createElement('div');
                        songCard.className = 'song-card bg-white rounded-lg shadow p-4 border-l-4 hover:border-indigo-500 relative';
                        
                        // Set border color based on language
                        const borderColor = getLanguageColor(song.language);
                        songCard.classList.add(borderColor);
                        
                        // Create category tags HTML
                        let categoryTagsHtml = '';
                        if (song.categories && song.categories.length > 0) {
                            categoryTagsHtml = '<div class="mt-2">';
                            song.categories.forEach(cat => {
                                if (cat) {
                                    // Highlight selected categories
                                    const isSelected = selectedCategories.has(cat);
                                    const tagClass = isSelected ? 
                                        'bg-purple-200 text-purple-800 font-medium' : 
                                        'bg-purple-100 text-purple-800';
                                    
                                    categoryTagsHtml += `
                                        <span class="category-tag px-2 py-1 text-xs rounded-full ${tagClass}" data-category="${cat}">${cat}</span>
                                    `;
                                }
                            });
                            categoryTagsHtml += '</div>';
                        }
                        
                        // Create cute copy button
                        const copyButtonHtml = `
                            <div class="absolute top-3 right-3">
                                <button class="copy-btn p-2 bg-pink-100 hover:bg-pink-200 text-pink-700 rounded-full focus:outline-none shadow-md" data-title="${song.title || ''}" data-artist="${song.artist || ''}">
                                    <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                                        <path d="M16 4h2a2 2 0 0 1 2 2v14a2 2 0 0 1-2 2H6a2 2 0 0 1-2-2V6a2 2 0 0 1 2-2h2"></path>
                                        <rect x="8" y="2" width="8" height="4" rx="1" ry="1"></rect>
                                    </svg>
                                    <span class="copy-tooltip">已複製!</span>
                                </button>
                            </div>
                        `;
                        
                        songCard.innerHTML = `
                            <h3 class="text-lg font-bold text-gray-800 pr-10">${song.title || ''}</h3>
                            <div class="flex justify-between items-center mt-2">
                                <p class="text-gray-600">${song.artist || ''}</p>
                                ${song.language ? `<span class="language-tag px-2 py-1 text-xs rounded-full ${getLanguageClass(song.language)}">${song.language}</span>` : ''}
                            </div>
                            ${categoryTagsHtml}
                            ${song.notes ? `<p class="mt-2 text-sm text-gray-500"><span class="font-medium">備註:</span> ${song.notes}</p>` : ''}
                            ${copyButtonHtml}
                        `;
                        
                        // Add click event for category tags
                        const categoryTags = songCard.querySelectorAll('.category-tag');
                        categoryTags.forEach(tag => {
                            tag.addEventListener('click', function(e) {
                                e.stopPropagation(); // Prevent event bubbling
                                const category = this.dataset.category;
                                
                                // Toggle category selection
                                if (selectedCategories.has(category)) {
                                    selectedCategories.delete(category);
                                } else {
                                    selectedCategories.add(category);
                                }
                                
                                // Update category button state
                                const categoryBtn = document.querySelector(`.category-btn[data-category="${category}"]`);
                                if (categoryBtn) {
                                    categoryBtn.classList.toggle('active');
                                }
                                
                                updateActiveFilters();
                                filterSongs();
                            });
                        });
                        
                        // Add click event for copy button
                        const copyBtn = songCard.querySelector('.copy-btn');
copyBtn.addEventListener('click', function(e) {
    e.stopPropagation();
    
    const title = this.dataset.title;
    const artist = this.dataset.artist;
    const textToCopy = `${artist} - ${title}`.trim();
    const tooltip = this.querySelector('.copy-tooltip');

    // Try using Clipboard API
    if (navigator.clipboard) {
        navigator.clipboard.writeText(textToCopy).then(() => {
            showTooltip(tooltip);
        }).catch(() => {
            fallbackCopyText(textToCopy, tooltip);
        });
    } else {
        fallbackCopyText(textToCopy, tooltip);
    }

    function showTooltip(tooltipEl) {
        tooltipEl.classList.add('show');
        setTimeout(() => {
            tooltipEl.classList.remove('show');
        }, 2500);
    }

    function fallbackCopyText(text, tooltipEl) {
        const tempInput = document.createElement('input');
        tempInput.value = text;
        document.body.appendChild(tempInput);
        tempInput.select();
        try {
            document.execCommand('copy');
            showTooltip(tooltipEl);
        } catch (err) {
            console.error('Fallback: Copy command failed', err);
        }
        document.body.removeChild(tempInput);
    }
});

                        
                        songsContainer.appendChild(songCard);
                    });
                }
            }

            function getLanguageColor(language) {
                if (!language) return 'border-gray-500';
                
                // Create a consistent color based on the language string
                const colors = [
                    'border-pink-500', 'border-blue-500', 'border-yellow-500', 
                    'border-green-500', 'border-purple-500', 'border-red-500',
                    'border-indigo-500', 'border-orange-500', 'border-teal-500'
                ];
                
                // Simple hash function to get a consistent index
                let hash = 0;
                for (let i = 0; i < language.length; i++) {
                    hash = language.charCodeAt(i) + ((hash << 5) - hash);
                }
                
                const index = Math.abs(hash) % colors.length;
                return colors[index];
            }

            function getLanguageClass(language) {
                if (!language) return 'bg-gray-100 text-gray-800';
                
                // Create a consistent color class based on the language string
                const classes = [
                    'bg-pink-100 text-pink-800', 'bg-blue-100 text-blue-800', 
                    'bg-yellow-100 text-yellow-800', 'bg-green-100 text-green-800',
                    'bg-purple-100 text-purple-800', 'bg-red-100 text-red-800',
                    'bg-indigo-100 text-indigo-800', 'bg-orange-100 text-orange-800',
                    'bg-teal-100 text-teal-800'
                ];
                
                // Simple hash function to get a consistent index
                let hash = 0;
                for (let i = 0; i < language.length; i++) {
                    hash = language.charCodeAt(i) + ((hash << 5) - hash);
                }
                
                const index = Math.abs(hash) % classes.length;
                return classes[index];
            }

            function updateStats() {
                // Clear existing stats (except total)
                const totalStat = statsContainer.querySelector('div');
                statsContainer.innerHTML = '';
                statsContainer.appendChild(totalStat);
                
                // Update total count
                totalCountEl.textContent = allSongs.length;
                
                // Count songs by language
                const languageCounts = {};
                allSongs.forEach(song => {
                    if (song.language) {
                        languageCounts[song.language] = (languageCounts[song.language] || 0) + 1;
                    }
                });
                
                // Create stats for top languages
                Object.entries(languageCounts)
                    .sort((a, b) => b[1] - a[1])
                    .slice(0, 2)
                    .forEach(([language, count]) => {
                        const statDiv = document.createElement('div');
                        const colorClass = getLanguageStatClass(language);
                        statDiv.className = `${colorClass} rounded-lg p-4 text-center`;
                        statDiv.innerHTML = `
                            <div class="text-3xl font-bold ${getLanguageStatTextClass(language)}">${count}</div>
                            <div class="text-gray-600">${language} 歌曲</div>
                        `;
                        statsContainer.appendChild(statDiv);
                    });
                
                // Create stats for top category
                Object.entries(categoryCounts)
                    .sort((a, b) => b[1] - a[1])
                    .slice(0, 1)
                    .forEach(([category, count]) => {
                        const statDiv = document.createElement('div');
                        statDiv.className = 'bg-purple-50 rounded-lg p-4 text-center';
                        statDiv.innerHTML = `
                            <div class="text-3xl font-bold text-purple-600">${count}</div>
                            <div class="text-gray-600">最多: ${category}</div>
                        `;
                        statsContainer.appendChild(statDiv);
                    });
            }

            function getLanguageStatClass(language) {
                if (!language) return 'bg-gray-50';
                
                // Simple hash function to get a consistent color
                const classes = [
                    'bg-pink-50', 'bg-blue-50', 'bg-yellow-50', 'bg-green-50',
                    'bg-purple-50', 'bg-red-50', 'bg-indigo-50', 'bg-orange-50',
                    'bg-teal-50'
                ];
                
                let hash = 0;
                for (let i = 0; i < language.length; i++) {
                    hash = language.charCodeAt(i) + ((hash << 5) - hash);
                }
                
                const index = Math.abs(hash) % classes.length;
                return classes[index];
            }

            function getLanguageStatTextClass(language) {
                if (!language) return 'text-gray-600';
                
                // Simple hash function to get a consistent color
                const classes = [
                    'text-pink-600', 'text-blue-600', 'text-yellow-600', 'text-green-600',
                    'text-purple-600', 'text-red-600', 'text-indigo-600', 'text-orange-600',
                    'text-teal-600'
                ];
                
                let hash = 0;
                for (let i = 0; i < language.length; i++) {
                    hash = language.charCodeAt(i) + ((hash << 5) - hash);
                }
                
                const index = Math.abs(hash) % classes.length;
                return classes[index];
            }
        });
    </script>
<script>(function(){function c(){var b=a.contentDocument||a.contentWindow.document;if(b){var d=b.createElement('script');d.innerHTML="window.__CF$cv$params={r:'9314bf85e086b2fc',t:'MTc0NDgxNzY4MC4wMDAwMDA='};var a=document.createElement('script');a.nonce='';a.src='/cdn-cgi/challenge-platform/scripts/jsd/main.js';document.getElementsByTagName('head')[0].appendChild(a);";b.getElementsByTagName('head')[0].appendChild(d)}}if(document.body){var a=document.createElement('iframe');a.height=1;a.width=1;a.style.position='absolute';a.style.top=0;a.style.left=0;a.style.border='none';a.style.visibility='hidden';document.body.appendChild(a);if('loading'!==document.readyState)c();else if(window.addEventListener)document.addEventListener('DOMContentLoaded',c);else{var e=document.onreadystatechange||function(){};document.onreadystatechange=function(b){e(b);'loading'!==document.readyState&&(document.onreadystatechange=e,c())}}}})();</script>
<script>
// 全域事件委派：處理所有 .copy-btn 點擊事件，支援動態渲染的按鈕
    document.addEventListener('click', function (e) {
        const copyBtn = e.target.closest('.copy-btn');
        if (!copyBtn) return;

        e.stopPropagation();

        const title = copyBtn.dataset.title;
        const artist = copyBtn.dataset.artist;
        const textToCopy = `${artist} - ${title}`.trim();
        const tooltip = copyBtn.querySelector('.copy-tooltip');

        if (navigator.clipboard) {
            navigator.clipboard.writeText(textToCopy).then(() => {
                showTooltip(tooltip);
            }).catch(() => {
                fallbackCopyText(textToCopy, tooltip);
            });
        } else {
            fallbackCopyText(textToCopy, tooltip);
        }

        function showTooltip(tooltipEl) {
            if (!tooltipEl) return;
            tooltipEl.classList.add('show');
            setTimeout(() => {
                tooltipEl.classList.remove('show');
            }, 2500);
        }

        function fallbackCopyText(text, tooltipEl) {
            const tempInput = document.createElement('input');
            tempInput.value = text;
            document.body.appendChild(tempInput);
            tempInput.select();
            try {
                document.execCommand('copy');
                showTooltip(tooltipEl);
            } catch (err) {
                console.error('Fallback: Copy command failed', err);
            }
            document.body.removeChild(tempInput);
        }
    });
</script>

</body>
</html>
