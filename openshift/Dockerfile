FROM registry.access.redhat.com/ubi8/dotnet-60:6.0-1

#these commands can be updated to accomodate compiling the artifact in the container for ease of use
#fix according to the ci/cd pipeline needs  
ADD app/ .

RUN dotnet publish -c Release

CMD ["dotnet", "./bin/Release/net6.0/publish/app.dll"]
