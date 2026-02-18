function loadData() {
            const month = monthSelect.value;
            workers.forEach((worker, index) => {
                const key = worker_${index}_month_${month};
                const data = JSON.parse(localStorage.getItem(key)) || { worked: [], compensatory: [], available: 0 };
                renderWorker(worker, index, data);
            });
        }

        function renderWorker(name, index, data) {
            const section = document.createElement('div');
            section.className = 'worker-section';
            section.innerHTML = 
                <h3>${name}</h3>
                <p>Compensatorios disponibles: <span class="compensatory-counter">${data.available}</span></p>
                <div class="day-grid" id="grid-${index}">
                    ${Array.from({length: 31}, (_, i) => <div class="day-box" data-day="${i+1}">${i+1}</div>).join('')}
                </div>
            ;
            container.appendChild(section);

            const grid = section.querySelector(#grid-${index});
            const boxes = grid.querySelectorAll('.day-box');
            const counter = section.querySelector('.compensatory-counter');

            // Aplicar estado inicial
            data.worked.forEach(day => boxes[day-1].classList.add('worked'));
            data.compensatory.forEach(day => boxes[day-1].classList.add('compensatory'));

            boxes.forEach(box => {
                box.addEventListener('click', () => {
                    const day = parseInt(box.dataset.day);
                    if (box.classList.contains('worked')) {
                        box.classList.remove('worked');
                        data.worked = data.worked.filter(d => d !== day);
                        data.available = Math.max(0, data.available - 1);
                    } else if (box.classList.contains('compensatory')) {
                        box.classList.remove('compensatory');
                        data.compensatory = data.compensatory.filter(d => d !== day);
                        data.available += 1;
                    } else {
                        // Marcar como trabajado y ganar compensatorio si posible
                        box.classList.add('worked');
                        data.worked.push(day);
                        data.available += 1;
                        // Automáticamente marcar un compensatorio si hay disponibles
                        const availableBox = Array.from(boxes).find(b => !b.classList.contains('worked') && !b.classList.contains('compensatory'));
                        if (availableBox && data.available > 0) {
                            availableBox.classList.add('compensatory');
                            data.compensatory.push(parseInt(availableBox.dataset.day));
                            data.available -= 1;
                        }
                    }
                    counter.textContent = data.available;
                    saveData(index, data);
                });
            });
        }

        function saveData(index, data) {
            const month = monthSelect.value;
            const key = worker_${index}_month_${month};
            localStorage.setItem(key, JSON.stringify(data));
        }

        monthSelect.addEventListener('change', () => {
            container.innerHTML = '';
            loadData();
        });

        loadData(); // Cargar datos iniciales
    </script>
</body>
</html>
