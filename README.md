# Vikas-sharma-
Vikas sharma 
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>SaaS Invoice Generator</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">

    <!-- Bootstrap 5 CDN -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">

    <!-- Google Font -->
    <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;600&display=swap" rel="stylesheet">

    <style>
        body {
            background: #f4f6f9;
            font-family: 'Poppins', sans-serif;
        }

        .card {
            border-radius: 15px;
        }

        .invoice-preview {
            background: white;
            padding: 30px;
            border-radius: 10px;
        }

        .logo-preview {
            max-height: 80px;
            margin-bottom: 10px;
        }

        .btn-primary {
            background: linear-gradient(135deg, #4e73df, #224abe);
            border: none;
        }

        .btn-success {
            background: linear-gradient(135deg, #1cc88a, #13855c);
            border: none;
        }
    </style>
</head>
<body>

<div class="container py-5">
    <div class="row g-4">

        <!-- ================= FORM SECTION ================= -->
        <div class="col-lg-5">
            <div class="card shadow p-4">
                <h4 class="mb-3">Invoice Details</h4>

                <!-- Business Info -->
                <input type="text" id="businessName" class="form-control mb-3" placeholder="Business Name">

                <input type="file" id="logoUpload" class="form-control mb-3" accept="image/*">

                <input type="text" id="clientName" class="form-control mb-3" placeholder="Client Name">

                <div class="row">
                    <div class="col">
                        <input type="text" id="invoiceNumber" class="form-control mb-3" placeholder="Invoice #">
                    </div>
                    <div class="col">
                        <input type="date" id="invoiceDate" class="form-control mb-3">
                    </div>
                </div>

                <!-- Items -->
                <h6>Items</h6>
                <table class="table table-sm" id="itemTable">
                    <thead>
                        <tr>
                            <th>Item</th>
                            <th>Price</th>
                            <th>Qty</th>
                            <th></th>
                        </tr>
                    </thead>
                    <tbody></tbody>
                </table>

                <button class="btn btn-sm btn-primary mb-3" onclick="addItem()">+ Add Item</button>

                <!-- Tax & Discount -->
                <input type="number" id="tax" class="form-control mb-2" placeholder="Tax (%)">
                <input type="number" id="discount" class="form-control mb-3" placeholder="Discount (%)">

                <button class="btn btn-success w-100" onclick="generateInvoice()">Generate Invoice</button>
            </div>
        </div>

        <!-- ================= PREVIEW SECTION ================= -->
        <div class="col-lg-7">
            <div class="card shadow p-4">
                <div id="invoice" class="invoice-preview">

                    <div class="d-flex justify-content-between">
                        <div>
                            <img id="logoPreview" class="logo-preview d-none"/>
                            <h3 id="previewBusinessName">Your Business</h3>
                        </div>
                        <div class="text-end">
                            <h5>Invoice</h5>
                            <p id="previewInvoiceNumber"></p>
                            <p id="previewDate"></p>
                        </div>
                    </div>

                    <hr>

                    <p><strong>Billed To:</strong> <span id="previewClientName"></span></p>

                    <table class="table mt-3">
                        <thead class="table-light">
                            <tr>
                                <th>Item</th>
                                <th>Price</th>
                                <th>Qty</th>
                                <th>Total</th>
                            </tr>
                        </thead>
                        <tbody id="previewItems"></tbody>
                    </table>

                    <div class="text-end">
                        <p>Subtotal: $<span id="subtotal">0.00</span></p>
                        <p>Tax: $<span id="taxAmount">0.00</span></p>
                        <p>Discount: -$<span id="discountAmount">0.00</span></p>
                        <h5>Total: $<span id="grandTotal">0.00</span></h5>
                    </div>

                </div>

                <button class="btn btn-primary mt-3 w-100" onclick="downloadPDF()">Download as PDF</button>
            </div>
        </div>

    </div>
</div>

<!-- ================= JAVASCRIPT ================= -->

<script src="https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.10.1/html2pdf.bundle.min.js"></script>

<script>
    // Add Item Row
    function addItem() {
        const table = document.querySelector("#itemTable tbody");

        const row = document.createElement("tr");
        row.innerHTML = `
            <td><input type="text" class="form-control item-name"></td>
            <td><input type="number" class="form-control item-price" value="0"></td>
            <td><input type="number" class="form-control item-qty" value="1"></td>
            <td><button class="btn btn-sm btn-danger" onclick="this.closest('tr').remove()">X</button></td>
        `;

        table.appendChild(row);
    }

    // Generate Invoice
    function generateInvoice() {

        document.getElementById("previewBusinessName").innerText =
            document.getElementById("businessName").value;

        document.getElementById("previewClientName").innerText =
            document.getElementById("clientName").value;

        document.getElementById("previewInvoiceNumber").innerText =
            "Invoice #: " + document.getElementById("invoiceNumber").value;

        document.getElementById("previewDate").innerText =
            "Date: " + document.getElementById("invoiceDate").value;

        const items = document.querySelectorAll("#itemTable tbody tr");
        const previewItems = document.getElementById("previewItems");
        previewItems.innerHTML = "";

        let subtotal = 0;

        items.forEach(row => {
            const name = row.querySelector(".item-name").value;
            const price = parseFloat(row.querySelector(".item-price").value);
            const qty = parseFloat(row.querySelector(".item-qty").value);
            const total = price * qty;

            subtotal += total;

            previewItems.innerHTML += `
                <tr>
                    <td>${name}</td>
                    <td>$${price.toFixed(2)}</td>
                    <td>${qty}</td>
                    <td>$${total.toFixed(2)}</td>
                </tr>
            `;
        });

        const taxPercent = parseFloat(document.getElementById("tax").value) || 0;
        const discountPercent = parseFloat(document.getElementById("discount").value) || 0;

        const taxAmount = subtotal * (taxPercent / 100);
        const discountAmount = subtotal * (discountPercent / 100);
        const grandTotal = subtotal + taxAmount - discountAmount;

        document.getElementById("subtotal").innerText = subtotal.toFixed(2);
        document.getElementById("taxAmount").innerText = taxAmount.toFixed(2);
        document.getElementById("discountAmount").innerText = discountAmount.toFixed(2);
        document.getElementById("grandTotal").innerText = grandTotal.toFixed(2);
    }

    // Logo Upload Preview
    document.getElementById("logoUpload").addEventListener("change", function() {
        const reader = new FileReader();
        reader.onload = function(e) {
            const logo = document.getElementById("logoPreview");
            logo.src = e.target.result;
            logo.classList.remove("d-none");
        }
        reader.readAsDataURL(this.files[0]);
    });

    // Download PDF
    function downloadPDF() {
        const element = document.getElementById("invoice");

        html2pdf().from(element).set({
            margin: 0.5,
            filename: 'invoice.pdf',
            html2canvas: { scale: 2 },
            jsPDF: { unit: 'in', format: 'letter', orientation: 'portrait' }
        }).save();
    }

    // Initialize with one item row
    addItem();
</script>

</body>
</html>
