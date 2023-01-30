FROM openjdk:11
# Build Main.java
# TODO : in next steps (not now)

FROM openjdk:11-jre
# Copy resource from previous stage
COPY --from=0 /usr/src/Main.class .
# Run java code with the JRE
# TODO : in next steps (not now)

#FROM openjdk:11
#COPY . /usr/src/myapp
#WORKDIR /usr/src/myapp
#RUN javac Main.java
#CMD ["java", "Main"]