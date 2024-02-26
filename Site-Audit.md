The following script can be used to scan all websites in "sites.txt".
A summary will be emailed to "EMAIL_TO".

```
#!/bin/bash

#
# site-audit.sh
#
# Checks all the sites given.
#

EMAIL_TO="testing@okmg.com"  # Space-separated emails.
EMAIL_FROM="Site Audit <no-reply@ses.okmg.com>"
EMAIL_SUBJECT="Daily Website Audit"
EMAIL_REGION="eu-west-1"
SITE_LIST="/home/bitnami/scripts/sites.txt"

#
# Main code.
#

function func_send_email {
  local text="$1"
  local html="$2"

  aws ses send-email \
    --to "$EMAIL_TO" \
    --subject "$EMAIL_SUBJECT" \
    --from "$EMAIL_FROM" \
    --text "$text" \
    --html "$html" \
    --region "$EMAIL_REGION"
}

function func_get_url_status {
  curl --write-out %{http_code} --silent --output /dev/null "$1"
}

function func_main {

  local successfulUrls=()
  local successfulStatuses=()
  local erroredUrls=()
  local erroredStatuses=()

  # Read all lines.
  while read -r line || [[ -n "$line" ]]; do

    # Check if line exists, and doesn't begin with '#'.
    if [[ ! -z "$line" ]] && [[ "$line" != \#* ]]; then
      local url="$line"

      # Get status of the URL.
      echo "Checking \"$line\""
      local status=$(func_get_url_status "$line")

      # If the status is weird, add it to list of errored urls.
      if [[ "$status" -ne 200 ]] && [[ "$status" -ne 301 ]] && [[ "$status" -ne 302 ]]; then
        erroredUrls+=("$url")
        erroredStatuses+=("$status")
      else
        successfulUrls+=("$url")
        successfulStatuses+=("$status")
      fi

    fi
  done < "$SITE_LIST"

  # Get email contents.
  local email_text="The following URLs failed:
$(for i in ${!erroredUrls[@]}; do echo "${erroredUrls[$i]} - ${erroredStatuses[$i]}"; done)

The following URLs succeeded:
$(for i in ${!successfulUrls[@]}; do echo "${successfulUrls[$i]} - ${successfulStatuses[$i]}"; done)"

  # Get email contents, HTML version.
  local email_html="<p>The following URLs failed:</p>
<table>
<tbody>
$(for i in ${!erroredUrls[@]}; do echo "<tr><td>${erroredUrls[$i]}</td><td>${erroredStatuses[$i]}</td></tr>"; done)
</tbody>
</table>
<p>The following URLs succeeded:</p>
<table>
<tbody>
$(for i in ${!successfulUrls[@]}; do echo "<tr><td>${successfulUrls[$i]}</td><td>${successfulStatuses[$i]}</td></tr>"; done)
</tbody>
</table>"

  # Send email.
  echo "Sending email"
  func_send_email "$email_text" "$email_html"

  exit 0
}

func_main "$@"
```

Example of "sites.txt":
```
# OKMG
http://okmg.com
https://okmg.com
http://www.okmg.com
https://www.okmg.com
```