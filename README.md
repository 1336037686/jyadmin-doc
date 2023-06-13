"# jyadmin-doc" 





docker build -f Dockerfile -t docsify/jyadmin-doc .
docker run -itp 3030:3030 --name=docsify -v $(pwd):/docs docsify/jyadmin-doc
