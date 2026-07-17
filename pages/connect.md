---
title: Quaker Connections
layout: page
permalink: /connect/
---

<!-- Linking Vis.js map software library -->
<script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/vis-network/10.1.0/standalone/umd/vis-network.min.js"></script>


<style>
  #network-container {
    width: 100%;
    height: 550px;
    border: 1px solid #dee2e6;
    border-radius: 6px;
    background-color: #fdfdfd;
  }
  .vis-network { cursor: pointer; }
</style>

<p class="lead">This graph showcases the different ways the Quakers in this collection connect to each other. From friends, to brothers, to biographers. Click, drag, or zoom into the diagram to explore each different relationships and their history. Select any connection line or portrait to display their historical context and botanical records in the sidebar.</p>

<div style="display: flex; gap: 20px; flex-wrap: wrap; margin-top: 15px;">
  <!-- Map -->
  <div style="flex: 2; min-width: 320px;">
    <div id="network-container"></div>
  </div>
  <!-- Sidebar Panel -->
  <div style="flex: 1; min-width: 260px; border: 1px solid #dee2e6; border-radius: 6px; padding: 20px; background: #ffffff; height: fit-content; max-height: 550px; overflow-y: auto;">
    <h3 id="panel-title" style="margin-top:0;">Select an Item</h3>
    <h6 id="panel-sub" style="color: #e51e1e; margin-bottom: 15px; font-weight: normal; text-transform: uppercase; font-size: 0.8rem;"></h6>

    <!-- Container for the Pop-up Portrait Image -->
  <div id="panel-img-container" style="display: none; margin-bottom: 15px; text-align: center;">
    <img id="panel-img" src="" alt="Portrait Image" style="max-width: 100%; height: auto; border-radius: 4px; border: 1px solid #dee2e6; box-shadow: 0 2px 4px rgba(0,0,0,0.05);">
    <p id="panel-imag-desc" style="font-size: 0.85rem; color: #6c757d; margin-top: 5px; font-style: italic;"></p>
  </div>

  <p id="panel-desc">Click on a person, institution, or relationship line in the web to view historical narratives, family connections, and plant ties.</p>

  <!-- Container for Source & Meta Information -->
  <div id="panel-source-container" style="display: none; margin-top: 20px; padding-top: 15px; border-top: 1px dashed #dee2e6; font-size: 0.85rem; color: #495057;">
    <strong>Image Date:</strong> <span id="panel-imag-date"></span><br>
    <strong style="display:block; margin-top: 5px;">Collection & Source Info:</strong>
    <span id="panel-source-info"></span>
  </div>
</div>

<script type="text/javascript">

// Relationship registry
  const relationshipRegistry = {
    {% for row in site.data.quakerconnects %}
    "{{ row.relationship_id }}": {
      title: "{{ row.person_1 }} & {{ row.person_2 }}",
      sub: "Connection: {{ row.connection_type }} | Plant: {{ row.associated_plant }}",
      desc: {{ row.historical_context | jsonify }}
    }{% unless forloop.last %},{% endunless %}
    {% endfor %}
  };

// Get information for the people information
  const nodeRegistry = {
    {% for row in site.data.people %}
    {{ row.person_name | strip | jsonify }}: {
      title: "{{ row.person_name }}",
      sub: "Date of birth: {{ row.dob }}",
      desc: {% if row.description and row.description != "" %}{{ row.description | jsonify }}{% else %}"No description available for this item."{% endif %},
      image: "{{ '/assets/img/' | relative_url }}{{ row.image }}",
      has_image: {% if row.image and row.image != "" %}true{% else %}false{% endif %},
      imag_desc: {{ row.imag_des | jsonify }},
      imag_date: {{ row.imag_date | jsonify }},
      // Maps to column H: "Collection & Source info" from your spreadsheet
      source_info: {{ row['Collection & Source info'] | jsonify }}
    }{% unless forloop.last %},{% endunless %}
    {% endfor %}
  };

// Displays the image in a circle 
  const canvasNodes = new vis.DataSet([
    {% for row in site.data.people %}
    { 
      id: {{ row.person_name | strip | jsonify }}, 
      label: "{{ row.person_name }}", 
      group: "{{ row.group }}",
      {% if row.image and row.image != "" %}
      image: "{{ '/assets/img/' | relative_url }}{{ row.image }}",
      shape: "circularImage"
      {% else %}
      shape: "{% if row.group == 'institution' %}square{% else %}dot{% endif %}"
      {% endif %}
    }{% unless forloop.last %},{% endunless %}
    {% endfor %}
  ]);


  // Get information for the connections
  const canvasEdges = new vis.DataSet([
    {% for row in site.data.quakerconnects %}
    { 
      id: "{{ row.relationship_id }}", 
      from: {{ row.person_1 | strip | jsonify }}, 
      to: {{ row.person_2 | strip | jsonify }}, 
      label: {{ row.connection_type | strip | jsonify }} 
    }{% unless forloop.last %},{% endunless %}
    {% endfor %}
  ]);

  // Building the network grid canvas layout
  const ctx = document.getElementById('network-container');
  const chart = new vis.Network(ctx, { nodes: canvasNodes, edges: canvasEdges }, {
    nodes: { 
      size: 28, 
      font: { size: 12, face: 'Arial' }, 
      borderWidth: 2,
      shadow: true
    },
    edges: { 
      width: 2, 
      color: '#b0bec5', 
      font: { size: 10, color: '#546e7a', align: 'horizontal' }, 
      arrows: { to: false } 
    },
    groups: {
      networked: { color: { background: '#e3f2fd', border: '#1e88e5' } },
      institution: { color: { background: '#ede7f6', border: '#5e35b1' }, size: 32 }
    }
  });

  // Click and get information
  chart.on("click", function (evt) {
    // Check nodes first
    if (evt.nodes && evt.nodes.length > 0) {
      const selectedNodeId = evt.nodes[0]; // Get the specific node ID string
      const nodeData = nodeRegistry[selectedNodeId];
      
      if (nodeData) {
        document.getElementById('panel-title').innerText = nodeData.title;
        document.getElementById('panel-sub').innerText = nodeData.sub;
        document.getElementById('panel-desc').innerText = nodeData.desc;

        // Adding the images into the side pannel
        if (nodeData.has_image) {
          document.getElementById('panel-img').src = nodeData.image;
          document.getElementById('panel-imag-desc').innerText = nodeData.imag_desc || "";
          document.getElementById('panel-img-container').style.display = "block";
        } else {
          document.getElementById('panel-img-container').style.display = "none";
        }

        // Adding image description and date 
        if (nodeData.source_info || nodeData.imag_date) {
          document.getElementById('panel-imag-date').innerText = nodeData.imag_date || "Unknown";
          document.getElementById('panel-source-info').innerText = nodeData.source_info || "No source provided.";
          document.getElementById('panel-source-container').style.display = "block";
        } else {
          document.getElementById('panel-source-container').style.display = "none";
        }
      }
    } 
    // If no node was clicked, check if an edge was clicked
    else if (evt.edges && evt.edges.length > 0) {
      const selectedEdgeId = evt.edges[0]; // Get the specific edge ID string
      const edgeData = relationshipRegistry[selectedEdgeId];
      
      if (edgeData) {
        document.getElementById('panel-title').innerText = edgeData.title;
        document.getElementById('panel-sub').innerText = edgeData.sub;
        document.getElementById('panel-desc').innerText = edgeData.desc;
      }
    }
  });
