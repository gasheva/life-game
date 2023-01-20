FROM node:16.16.0-alpine as develop-stage
RUN mkdir -p /home/life-game
COPY . /home/life-game
WORKDIR /home/life-game
RUN npm install
EXPOSE 5173
CMD ["npm", "run", "dev"]