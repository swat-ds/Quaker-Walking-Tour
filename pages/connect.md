---
title: Quaker Connections
layout: page
permalink: /connect/
---

<!-- 1. FIXED: Correctly pathed link to download the Vis.js map software library -->
<script type="text/javascript" src="https://unpkg.com"></script>

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

<p class="lead">Click, drag, or zoom into the diagram to explore connections. Select any connection line or portrait to display their historical context and botanical records in the sidebar.</p>

<div style="display: flex; gap: 20px; flex-wrap: wrap; margin-top: 15px;">
  <!-- Left Side: Visual Map Diagram Canvas -->
  <div style="flex: 2; min-width: 320px;">
    <div id="network-container"></div>
  </div>
  <!-- Right Side: Sidebar Panel -->
  <div style="flex: 1; min-width: 260px; border: 1px solid #dee2e6; border-radius: 6px; padding: 20px; background: #ffffff; height: fit-content; max-height: 550px; overflow-y: auto;">
    <h3 id="panel-title" style="margin-top:0;">Select an Item</h3>
    <h6 id="panel-sub" style="color: #1e88e5; margin-bottom: 15px; font-weight: normal; text-transform: uppercase; font-size: 0.8rem;"></h6>
    <p id="panel-desc">Click on a person, institution, or relationship line in the web to view historical narratives, family connections, and plant ties.</p>
  </div>
</div>

<script type="text/javascript">

  // 2. FIXED: Re-added the relationship text registry so clicking lines actually displays text
  const relationshipRegistry = {
    {% for row in site.data.quakerconnects %}
    "{{ row.relationship_id }}": {
      title: "{{ row.person_1 }} & {{ row.person_2 }}",
      sub: "Connection: {{ row.connection_type }} | Plant: {{ row.associated_plant }}",
      desc: {{ row.historical_context | jsonify }}
    }{% unless forloop.last %},{% endunless %}
    {% endfor %}
  };

  const nodeRegistry = {
    {% for row in site.data.people %}
    "{{ row.person_name }}": {
      title: "{{ row.person_name }}",
      sub: "Type: {{ row.group }}",
      desc: "This historical entity is part of your botanical network map. Click on any of the connecting lines attached to them to read the full historical context."
    }{% unless forloop.last %},{% endunless %}
    {% endfor %}
  };

  // Generates your circles and portraits cleanly
  const canvasNodes = new vis.DataSet([
    {% for row in site.data.people %}
    { 
      id: "{{ row.person_name }}", 
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

  // 3. FIXED: Removed the duplicate, broken duplicate line loop array
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

  // Single click tracker to update dashboard panel text seamlessly
  chart.on("click", function (evt) {
    if (evt.nodes.length > 0) {
      const nodeData = nodeRegistry[evt.nodes];
      if (nodeData) {
        document.getElementById('panel-title').innerText = nodeData.title;
        document.getElementById('panel-sub').innerText = nodeData.sub;
        document.getElementById('panel-desc').innerText = nodeData.desc;
      }
    } else if (evt.edges.length > 0) {
      const edgeData = relationshipRegistry[evt.edges];
      if (edgeData) {
        document.getElementById('panel-title').innerText = edgeData.title;
        document.getElementById('panel-sub').innerText = edgeData.sub;
        document.getElementById('panel-desc').innerText = edgeData.desc;
      }
    }
  });
</script>
