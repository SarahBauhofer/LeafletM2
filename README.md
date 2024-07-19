# Leaflet
Mit diesem Code erhalten Mitarbeitende einer Firma in der Landwirtschaftstechnik die Möglichkeit, Kundenaufträge und ihre Auftragsdauer zu optimieren.

# Voraussetzungen
Server Webstorm Javascript Leaflet Schnittstelle zwischen Trello und Leaflet Traktor Icons herunterladen Zugriff Google Maps Reponsive Design

# Anfänge
1. Clone the repository: git clone https://github.com/paramsgit/leaflet

2. Leaflet Installieren https://leafletjs.com/download.html (Version 1.9.4)

3. Server installieren (javascript bibliothek)

4. Webstorm installieren

5. html Datei erstellen

# Anwendung
Die Karte öffnet sich nach erfolgreicher Anmeldung und fokussiert auf den aktuellen Standort des Benutzenden. Die Karte zeigt Traktor Icons, welche in der Farbe des jeweiligen Fälligkeitsdatums des Kundenauftrags gefärbt sind (grün = , gelb = , rot =, schwarz =). Wenn der Benutzende auf eines der Icons klickt, erhält er die Möglichkeit sich den Aufrag genauer anzusehen sowie sich eine Route zum Kunden berechnen zu lassen. Die Routenplanung erfolgt dann extern über GoogleMaps. Der Benutzende wird bei Klick auf den Button "Route planen" zu GoogleMaps weiteregeleitet. Über die Eingabemaske können die Kunden- und Auftragsdaten manuell eingetragen werden. Die entsprechenden Daten werden automatisch auf der Karte aktualisiert. Zusätzlich können bestehende Kunden bearbeitet werden.

# Bereitstellung
Der Code wird ausschließlich der Firma zur Verfügung gestellt. Mitarbeitende erhalten Informationen sowie Anmeldedaten von Ihrem Arbeitgeber. Die Nutzung ist sowohl über mobile Endgeräte als auch Computer möglich.

# Lizenz
Diese Prohjekt ist unter der Unlicense lizensiert. Siehe die LICENSE-Datei für weitere Detaile.

# Mitwirkung
Eine Mitwirkung ist ausschließlich den Beteiligten der Firma gestattet. Falls Mitwirkungen angegeben werden sollen/ wollen, muss hier die Zustimmung der Firma erfolgen und Kontakt aufgenommen werden.

# finaler Code
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Kundenkarte mit Info-Symbolen und Routenplaner</title>
    <!-- Lade Leaflet CSS -->
    <link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css" />
    <!-- Lade alertify CSS -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/AlertifyJS/1.13.1/css/alertify.min.css" />
    <!-- Lade alertify Theme CSS (optional) -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/AlertifyJS/1.13.1/css/themes/default.min.css" />
    <style>
        /* Stil für die Karte */
        #map {
            position: absolute;
            top: 0;
            left: 0;
            height: 100%;
            width: 100%;
            z-index: 1; /* Stellt sicher, dass die Karte im Vordergrund ist */
        }

        /* Stil für das Menüfeld */
        #menu {
            position: absolute;
            top: 10px;
            right: 10px;
            z-index: 2; /* Stellt sicher, dass das Menüfeld über der Karte liegt */
            background-color: rgba(255, 255, 255, 0.8);
            padding: 10px;
            border-radius: 5px;
        }

        /* Stil für die Eingabemaske */
        .customer-card {
            display: none; /* Verstecke die Eingabemaske standardmäßig */
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            z-index: 2; /* Stellt sicher, dass die Eingabemaske über der Karte liegt */
            background-color: white;
            padding: 20px;
            border-radius: 5px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
        }

        /* Stil für das Schließen-Symbol */
        .close-button {
            position: absolute;
            top: 10px;
            right: 10px;
            cursor: pointer;
        }

        /* Responsives Design */
        @media only screen and (max-width: 768px) {
            #menu {
                top: 70px; /* Anpassung für kleinere Bildschirme */
                right: 10px;
            }
        }
    </style>
</head>
<body>

<!-- Erstelle eine Container-Div für die Karte -->
<div id="map"></div>

<!-- Menüfeld mit der Option "Neuen Kunden anlegen" -->
<div id="menu">
    <button onclick="toggleCustomerForm()">Neuen Kunden anlegen</button>
</div>

<!-- Eingabemaske für neue Kundendaten -->
<div class="customer-card" id="customerCard">
    <span class="close-button" onclick="toggleCustomerForm()">X</span> <!-- Schließen-Symbol -->
    <h3 id="customerFormTitle">Neuen Kunden hinzufügen</h3>
    <label for="name">Name:</label><br>
    <input type="text" id="name" name="name" required><br>
    <label for="address">Adresse:</label><br>
    <input type="text" id="address" name="address" required><br>
    <label for="phone">Telefon:</label><br>
    <input type="text" id="phone" name="phone" required><br>
    <label for="email">E-Mail:</label><br>
    <input type="email" id="email" name="email" required><br>
    <label for="dueDate">Fälligkeitsdatum:</label><br>
    <input type="text" id="dueDate" name="dueDate" required><br>
    <label for="duration">Geschätzte Dauer (Tage):</label><br>
    <input type="number" id="duration" name="duration" required><br>
    <label for="orderDetails">Auftragsdetails:</label><br>
    <textarea id="orderDetails" name="orderDetails" rows="4" cols="40"></textarea><br>
    <button id="addCustomerButton" onclick="addOrUpdateCustomer()">Kunden hinzufügen</button>
    <button onclick="toggleCustomerForm()">Abbrechen</button> <!-- Abbrechen-Option -->
</div>

<!-- Lade Leaflet JavaScript -->
<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
<!-- Lade alertify JavaScript -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/AlertifyJS/1.13.1/alertify.min.js"></script>

<script>
    // Initialisiere die Karte
    var mymap = L.map('map').setView([51.1657, 10.4515], 6);

    // Füge die OpenStreetMap-Kartenschicht hinzu
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution: '&copy; OpenStreetMap contributors'
    }).addTo(mymap);

    // Icons definieren
    var tractorIconBlack = L.icon({
        iconUrl: 'black_icon.png',
        iconSize: [42, 42],
        iconAnchor: [21, 42],
        popupAnchor: [1, -34],
        tooltipAnchor: [16, -28],
        shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.7.1/images/marker-shadow.png',
        shadowSize: [42, 42]
    });
    var tractorIconGreen = L.icon({
        iconUrl: 'green_icon.png',
        iconSize: [42, 42],
        iconAnchor: [21, 42],
        popupAnchor: [1, -34],
        tooltipAnchor: [16, -28],
        shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.7.1/images/marker-shadow.png',
        shadowSize: [42, 42]
    });
    var tractorIconOrange = L.icon({
        iconUrl: 'orange_icon.png',
        iconSize: [42, 42],
        iconAnchor: [21, 42],
        popupAnchor: [1, -34],
        tooltipAnchor: [16, -28],
        shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.7.1/images/marker-shadow.png',
        shadowSize: [42, 42]
    });
    var tractorIconRed = L.icon({
        iconUrl: 'red_icon.png',
        iconSize: [42, 42],
        iconAnchor: [21, 42],
        popupAnchor: [1, -34],
        tooltipAnchor: [16, -28],
        shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.7.1/images/marker-shadow.png',
        shadowSize: [42, 42]
    });

    // Kunden mit ihren Standorten und Informationen
    var customers = [
        {
            name: "Landtechnik Müller Hans",
            location: [47.6595, 9.4783], // Koordinaten für Friedrichshafen
            address: "Am Seemooser Horn 20, 88045 Friedrichshafen",
            phone: "0123-456789 ",
            email: "mueller@landtechnik.de",
            order: "Traktor GPS-Installation",
            dueDate: "15.06.2024",
            duration: 3
        },
        {
            name: "Agrar Service Schmidt",
            location: [47.6607, 9.1751], // Koordinaten für Konstanz
            address: "Alfred-Wachtel-Straße 8, 78462 Konstanz",
            phone: "0234-567890",
            email: "schmidt@agrar.de",
            order: "Lenksystem-Upgrade",
            dueDate: "01.05.2024",
            duration: 2
        },
        {
            name: "Bauer Huber GmbH",
            location: [47.7238, 10.3121], // Koordinaten für Kempten
            address: "Bahnhofstraße 61, 87435 Kempten",
            phone: "03456-678901",
            email: "huber@bauernhof.de",
            order: "GPS-Antennenmontage",
            dueDate: "30.04.2024",
            duration: 1
        },
        {
            name: "Grünfeld AG",
            location: [47.7752, 9.6128], // Koordinaten für Ravensburg
            address: "Marienplatz 2, 88212 Ravensburg",
            phone: "0456-78901",
            email: "gruen@gruenfeld.ag",
            order: "Feldkartierung mit GPS",
            dueDate: "10.03.2024",
            duration: 5
        },
        {
            name: "Feldwerk KG",
            location: [48.1011, 9.7908], // Koordinaten für Biberach
            address: "Karlstraße 11, 88400 Biberach",
            phone: "0567-890123",
            email: "feld@feldwerk.kg",
            order: "Automatische Lenkassistenz",
            dueDate: "25.04.2024",
            duration: 4
        },

        {
            name: "Ackerguide",
            location: [48.93322, 8.96039], // Koordinaten für "Katharinenstraße 6, 71665 Vaihingen an der Enz"
            address: "Katharinenstraße 6, 71665 Vaihingen an der Enz"
        }
    ];

    function update_map() {
        console.log(customers);
        var customersStorage = customers; //JSON.parse(localStorage.getItem('customers'));
        //console.log(customersStorage);
        //var customerStorage = loadfilefromServer(url, key)
        //if (customersStorage === null) {
        //    customersStorage = customers;
        //}

        customersStorage.forEach(function(customer) {
            console.log("++++")
            console.log(customer)
            if ("marker" in customer) {
                console.log("TRUE")
                mymap.removeLayer(customer["marker"])
            }
            var today = new Date();
            console.log(customer.name);
            console.log(customer.phone);
            today = new Date(today.getFullYear(), today.getMonth(), today.getDate());

            // Marker für Kundenstandort hinzufügen
            if (customer.hasOwnProperty("dueDate")) {
                var due_data = customer.dueDate.split(".")
                var day = parseInt(due_data[0])
                var month = parseInt(due_data[1])
                var year = parseInt(due_data[2])

                var due_date = new Date(year, month - 1, day);
                var short_time = new Date(today.getFullYear(), today.getMonth(), today.getDate() + 14);
                var medium_time = new Date(today.getFullYear(), today.getMonth() + 2, today.getDate());

                if (short_time > due_date) {
                    var marker = L.marker(customer.location, {icon: tractorIconRed}).addTo(mymap);
                } else if (medium_time > due_date) {
                    var marker = L.marker(customer.location, {icon: tractorIconOrange}).addTo(mymap)
                } else {
                    var marker = L.marker(customer.location, {icon: tractorIconGreen}).addTo(mymap);
                }
            } else {
                var marker = L.marker(customer.location, {icon: tractorIconBlack}).addTo(mymap);
            }
            customer["marker"] = marker;
            var popupContent = "<b>" + customer.name + "</b><br>" +
                "Adresse: " + customer.address + "<br><br>" +
                "Telefon: " + customer.phone + "<br><br>" +
                "E-Mail: " + customer.email + "<br><br>" +
                "<button onclick=\"planRoute('" + customer.address + "')\">Route planen</button>" + "&nbsp;" +
                "<button onclick=\"viewOrder('" + customer.order + "', '" + customer.dueDate + "', " + customer.duration + ")\">Auftrag ansehen</button>" +
                "<button onclick=\"editCustomer('" + customer.name + "', '" + customer.address + "', '" + customer.phone + "', '" + customer.email + "', '" + customer.dueDate + "', " + customer.duration + ", '" + customer.order + "')\">Bearbeiten</button>";
            marker.bindPopup(popupContent);
        });
    }

    // Funktion zum Anzeigen der Eingabemaske
    function toggleCustomerForm() {
        var customerForm = document.getElementById("customerCard");
        if (customerForm.style.display === "") {
            customerForm.style.display = "block";
            document.getElementById("customerFormTitle").innerText = "Neuen Kunden hinzufügen";
            document.getElementById("addCustomerButton").innerText = "Kunden hinzufügen";
            clearInputFields(); // Leere die Eingabefelder, wenn das Formular geöffnet wird
        } else {
            customerForm.style.display = "";
        }
    }

    update_map()

    // drawIcon()
    // Funktion zum Leeren der Eingabefelder
    function clearInputFields() {
        document.getElementById("name").value = "";
        document.getElementById("address").value = "";
        document.getElementById("phone").value = "";
        document.getElementById("email").value = "";
        document.getElementById("dueDate").value = "";
        document.getElementById("duration").value = "";
        document.getElementById("orderDetails").value = "";
    }

    // Funktion zum Anzeigen der Auftragsdetails in einem Popup-Fenster
    function viewOrder(order, dueDate, duration) {
        var popupContent = "<div style='text-align: left; padding: 10px;'>" +
            "<h3 style='margin-bottom: 10px;'>Auftragsdetails</h3>" +
            "<p><b>Auftragsbezeichnung:</b> " + order + "</p>" +
            "<p><b>Fälligkeitsdatum:</b> " + dueDate + "</p>" +
            "<p><b>Geschätzte Dauer (Tage):</b> " + duration + "</p>" +
            "<button onclick=\"completeOrder('" + order + "')\">Erledigt</button>" +
            "</div>";
        alertify.alert('Auftragsdetails', popupContent).set({transition:'zoom'}).show();
    }

    // Funktion zum Bearbeiten eines bestehenden Kunden
    function editCustomer(name, address, phone, email, dueDate, duration, order) {
        document.getElementById("name").value = name;
        document.getElementById("address").value = address;
        document.getElementById("phone").value = phone;
        document.getElementById("email").value = email;
        document.getElementById("dueDate").value = dueDate;
        document.getElementById("duration").value = duration;
        document.getElementById("orderDetails").value = order;

        var customerForm = document.getElementById("customerCard");
        customerForm.style.display = "block";
        document.getElementById("customerFormTitle").innerText = "Kunde bearbeiten";
        document.getElementById("addCustomerButton").innerText = "Änderungen speichern";
    }

    // Funktion zum Hinzufügen oder Bearbeiten eines Kunden
    async function addOrUpdateCustomer() {
        var name = document.getElementById("name").value;
        var address = document.getElementById("address").value;
        var phone = document.getElementById("phone").value;
        var email = document.getElementById("email").value;
        var dueDate = document.getElementById("dueDate").value;
        var duration = document.getElementById("duration").value;
        var order = document.getElementById("orderDetails").value;

        // Kundenstandort auf der Karte markieren
        var result = await geocodeAddress(address);
        if (result[0]) {
            var new_location = [parseFloat(result[1][0]), parseFloat(result[1][1])];
            console.log("+++++++")
            console.log(new_location); // Corrected from location to new_location
            toggleCustomerForm(); // Eingabemaske schließen
        } else {
            alertify.error(result[1]); // Use result[1] instead of result.error
        }

        var new_customer = true;
        for (let element of customers) { // You can use let instead of const if you like
            console.log(element.name)
            console.log(new_location)
            if (name == element.name) {
                new_customer = false;
                console.log("EXIST")
                element.address = address;
                element.phone = phone;
                element.email = email;
                element.dueDate = dueDate;
                element.duration = duration;
                element.order = order;
                element.location = new_location;
            }
        }

        if (new_customer) {
            console.log("NEW")
            // Kundenobjekt erstellen
            var newCustomer = {
                name: name,
                address: address,
                phone: phone,
                email: email,
                dueDate: dueDate,
                duration: duration,
                order: order,
                location: new_location
            };
            customers.push(newCustomer);
        }

        console.log(customers);
        update_map()
    }

    async function fetchData(url) {
        const response = await fetch(url)
        const data = await response.json()
        return data;
    }

    // Funktion zum Geokodieren einer Adresse mit Nominatim
    async function geocodeAddress(address) {
        var url = "https://nominatim.openstreetmap.org/search?format=json&limit=1&q=" + encodeURIComponent(address);
        let return_value = await fetchData(url).then(data => {
            console.log("**")
            console.log(data.length)
            console.log(data)
            if (data && data.length > 0) {
                var lat = data[0].lat;
                var lon = data[0].lon;
                let value = [lat, lon];
                let success = true;
                return [success, value];
            } else {
                 let success = false;
                 let value = "Adresse nicht gefunden: " + address
                 return [success, value];
            }
        });
        console.log("++");
        console.log(return_value);
        return return_value;
    }

    // Funktion zum Markieren des aktuellen Standorts
    function markCurrentLocation(position) {
        var latitude = position.coords.latitude;
        var longitude = position.coords.longitude;
        var marker = L.marker([latitude, longitude]).addTo(mymap);
        marker.bindPopup("Dein Standort").openPopup();
        mymap.setView([latitude, longitude], 10); // Setze den Zoom-Level auf 10
    }

    // Funktion zum Abrufen der aktuellen Position
    function getLocation() {
        if (navigator.geolocation) {
            navigator.geolocation.getCurrentPosition(markCurrentLocation);
        } else {
            alert("Geolocation wird von diesem Browser nicht unterstützt.");
        }
    }

    // Funktion zum Planen einer Route
    function planRoute(address) {
        var url = "https://www.google.com/maps/dir/?api=1&destination=" + encodeURIComponent(address);
        window.open(url, '_blank');
    }

    // Funktion zum Abschließen eines Auftrags
    function completeOrder(order) {
        alertify.confirm('Auftrag erledigt', 'Auftrag "' + order + '" wurde als erledigt markiert. Er wird nun von der Karte entfernt.', function() {
            // Auftrag von der Karte entfernen
            var updatedCustomers = customers.filter(function(customer) {
                return customer.order !== order;
            });
            localStorage.setItem('customers', JSON.stringify(updatedCustomers)); // Aktualisierte Kundenliste speichern
            // setFileonserver(url, key, JSON.stringify(updatedCustomers)); )
            location.reload(); // Seite neu laden, um die Änderungen anzuzeigen
        }, function() {
            // User hat "Abbrechen" geklickt, keine weitere Aktion erforderlich
        });
    }

    // Standorterkennung aufrufen
    getLocation();

</script>

</body>
</html>
