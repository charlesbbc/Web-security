# IIS File Upload Configuration Abuse

Configuration on IIS servers using a web.config

This might include directives such as the following, which in this case allows JSON files to be served to users:

`<staticContent>
    <mimeMap fileExtension=".json" mimeType="application/json" />
</staticContent>`
