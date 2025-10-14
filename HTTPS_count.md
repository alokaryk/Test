ss -tlnp | grep :443  # For port 80 (HTTP); replace :80 with :443 for HTTPS
# Or total HTTP: ss -tunap | grep -E ':(80|443)' | wc -l
