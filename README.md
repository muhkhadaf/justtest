<!-- Validation Modal -->
    <div id="validationModal" class="fixed inset-0 bg-gray-600 bg-opacity-50 hidden overflow-y-auto h-full w-full z-50">
        <div class="relative top-20 mx-auto p-5 border w-96 shadow-lg rounded-md bg-white">
            <div class="mt-3">
                <div class="flex justify-between items-center mb-4">
                    <h3 class="text-lg leading-6 font-medium text-gray-900">Validasi Data HP</h3>
                    <button onclick="closeValidationModal()" class="text-gray-400 hover:text-gray-500">
                        <i class="fas fa-times"></i>
                    </button>
                </div>
                <form id="validationForm">
                    <input type="hidden" id="validation_id">
                    
                    <div class="mb-4">
                        <p class="text-sm text-gray-600 mb-1">Barcode:</p>
                        <p class="font-bold text-gray-800" id="validation_barcode_display">-</p>
                        <input type="hidden" id="validation_barcode">
                    </div>

                    <div class="mb-4">
                        <label class="block text-gray-700 text-sm font-bold mb-2">Status Validasi *</label>
                        <select id="validation_status" onchange="toggleValidationNotes()" class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline" required>
                            <option value="">-- Pilih Status --</option>
                            <option value="valid">Valid</option>
                            <option value="invalid">Tidak Valid</option>
                        </select>
                    </div>

                    <div id="validation_notes_container" class="mb-4 hidden">
                        <label class="block text-gray-700 text-sm font-bold mb-2">Keterangan / Notes *</label>
                        <textarea id="validation_notes" rows="3" class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline" placeholder="Masukkan alasan kenapa tidak valid..."></textarea>
                    </div>

                    <div class="flex justify-end gap-2 mt-4">
                        <button type="button" onclick="closeValidationModal()" class="px-4 py-2 bg-gray-300 text-gray-800 text-sm font-medium rounded-md shadow-sm hover:bg-gray-400 focus:outline-none focus:ring-2 focus:ring-gray-300">
                            Batal
                        </button>
                        <button type="button" onclick="saveValidation()" class="px-4 py-2 bg-blue-500 text-white text-sm font-medium rounded-md shadow-sm hover:bg-blue-600 focus:outline-none focus:ring-2 focus:ring-blue-300">
                            Simpan
                        </button>
                    </div>
                </form>
            </div>
        </div>
    </div>
