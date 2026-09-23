# Network Security Notes

The EdgeRouter separates the Airtel upstream network from the AstraHM LAN. Outbound NAT is enabled.

Before exposing services, implement and validate a formal EdgeOS WAN_LOCAL/WAN_IN policy, including established/related handling and management-plane restrictions.

Never commit passwords, SSH private keys, tokens, or API secrets. Keep credentials in a password manager.