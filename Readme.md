Perfect — you're building a multi-vendor MOP generator with support for:

✅ Cisco 902 (IOS-XE)

✅ Cisco 9010 (IOS-XR)

✅ Juniper (MX series)

✅ Alcatel (Nokia SR)

✅ ECI (possibly Apollo/Neptune platforms)

✅ Updated Web Page: Multi-Vendor MOP Generator (Cisco, Juniper, Alcatel, ECI)
Here’s the full standalone webpage with a dropdown for all 5 vendor types, and correct template generation logic for each.

💾 Save as multi-vendor-mop.html and open in your browser:
html
Copy
Edit
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Multi-Vendor MOP Generator</title>
  <style>
    body { font-family: Arial, sans-serif; background: #f8f8f8; padding: 25px; max-width: 900px; margin: auto; }
    label { display: block; margin-top: 15px; font-weight: bold; }
    input, select { padding: 6px; width: 340px; }
    button { margin-top: 25px; padding: 10px 25px; font-size: 16px; cursor: pointer; }
    textarea { width: 100%; height: 400px; margin-top: 20px; font-family: monospace; white-space: pre; }
  </style>
</head>
<body>

  <h1>🌐 Multi-Vendor XConnect MOP Generator</h1>

  <label for="vendor">Select Vendor:</label>
  <select id="vendor">
    <option value="cisco-902">Cisco 902 (IOS-XE)</option>
    <option value="cisco-9010">Cisco 9010 (IOS-XR)</option>
    <option value="juniper">Juniper</option>
    <option value="alcatel">Alcatel (Nokia SR)</option>
    <option value="eci">ECI</option>
  </select>

  <label for="interface">Interface:</label>
  <input type="text" id="interface" placeholder="e.g. Gig0/0/0 or xe-4/2/1" />

  <label for="unit_vlan">VLAN / Unit / Service ID:</label>
  <input type="text" id="unit_vlan" placeholder="e.g. 2589 or 3377" />

  <label for="description">Description:</label>
  <input type="text" id="description" placeholder="e.g. TO_DUMMY_1937568" />

  <label for="remote_ip">Remote PE IP:</label>
  <input type="text" id="remote_ip" placeholder="e.g. 182.19.97.187" />

  <label for="vcid">VC ID / PW ID:</label>
  <input type="text" id="vcid" placeholder="e.g. 21050863" />

  <button onclick="generateMOP()">Generate MOP</button>

  <textarea id="output" placeholder="Generated MOP will appear here..." readonly></textarea>

  <script>
    function generateMOP() {
      const vendor = document.getElementById("vendor").value;
      const iface = document.getElementById("interface").value.trim();
      const vlan = document.getElementById("unit_vlan").value.trim();
      const desc = document.getElementById("description").value.trim();
      const ip = document.getElementById("remote_ip").value.trim();
      const vcid = document.getElementById("vcid").value.trim();

      if (!iface || !vlan || !desc || !ip || !vcid) {
        alert("Please fill in all fields.");
        return;
      }

      let mop = "";

      if (vendor === "cisco-902") {
        mop = `========== Cisco 902 (IOS-XE) MOP ==========\n
interface ${iface}
 service instance ${vlan} ethernet
  description ${desc}
  encapsulation dot1q ${vlan}
  rewrite ingress tag pop 1 symmetric
  xconnect ${ip} ${vcid} encapsulation mpls\n
🔍 show run interface ${iface}
🔍 show xconnect all\n`;
      }

      else if (vendor === "cisco-9010") {
        mop = `========== Cisco 9010 (IOS-XR) MOP ==========\n
interface ${iface}.${vlan} l2transport
 description ${desc}
 encapsulation dot1q ${vlan}
 rewrite ingress tag pop 1 symmetric
 mtu 9018

l2vpn xconnect group EOMPLS_xconnect
 p2p ${vlan}_XCONN
  interface ${iface}.${vlan}
  neighbor ipv4 ${ip} pw-id ${vcid}
  neighbor ipv4 ${ip} pw-id ${vcid} pw-class VLAN\n
🔍 show l2vpn xconnect group EOMPLS_xconnect
🔍 show mpls l2transport vc interface ${iface}.${vlan}\n`;
      }

      else if (vendor === "juniper") {
        mop = `========== Juniper MOP ==========\n
set interfaces ${iface} unit ${vlan} apply-groups-except ARP_RE
set interfaces ${iface} unit ${vlan} description ${desc}
set interfaces ${iface} unit ${vlan} encapsulation vlan-ccc
set interfaces ${iface} unit ${vlan} vlan-id ${vlan}
set interfaces ${iface} unit ${vlan} input-vlan-map swap
set interfaces ${iface} unit ${vlan} input-vlan-map vlan-id ${vlan}
set interfaces ${iface} unit ${vlan} output-vlan-map swap
set protocols l2circuit neighbor ${ip} interface ${iface}.${vlan} virtual-circuit-id ${vcid}
set protocols l2circuit neighbor ${ip} interface ${iface}.${vlan} no-control-word
set protocols l2circuit neighbor ${ip} interface ${iface}.${vlan} mtu 9000\n
🔍 show configuration interfaces ${iface}
🔍 show configuration protocols l2circuit\n`;
      }

      else if (vendor === "alcatel") {
        mop = `========== Alcatel (Nokia SR) MOP ==========\n
configure service
 service ${vlan} customer 1 create
  description "${desc}"
  sap ${iface}:${vlan} create
  encapsulation dot1q ${vlan}
  exit
  spoke-sdp ${ip}:${vcid} create
   no shutdown
exit all\n
🔍 show service id ${vlan} detail
🔍 show service sdp-using\n`;
      }

      else if (vendor === "eci") {
        mop = `========== ECI MOP ==========\n
configure bridge ${vlan}
 port ${iface} vlan ${vlan}
 l2vpn enable
 pw peer ${ip} vc-id ${vcid}
 description ${desc}
 mtu 9000
 exit\n
🔍 show bridge ${vlan}
🔍 show pw status\n`;
      }

      document.getElementById("output").value = mop;
    }
  </script>
</body>
</html>
✅ Supported Vendor Config Styles
Vendor	Description Style
Cisco 902	IOS-XE service instance (ASR 902/903)
Cisco 9010	IOS-XR l2vpn xconnect group with .l2transport
Juniper	MX set interfaces + l2circuit
Alcatel/Nokia	SR configure service with SDP
ECI	CLI-style bridge domain with pseudowire

✅ Want to Export as .txt or Host Online?
I can help you:

Add a download button (Save as .txt)

Deploy it on GitHub Pages / Netlify for public access

Support bulk MOPs, or advanced templates (MPLS/VPLS/L3VPN)

Let me know what you want next!