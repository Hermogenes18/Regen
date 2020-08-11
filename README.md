# Regen
Juego regen --- utilizamos el motor de juegos HCGIL
nuestro código esta implementado bajo el patrón de arquitectura ECS (Entity Component System)

Los estilos de programación utilizados son : 
##Estilos
### Estilo things:
```C++
class VirusCreator
{
public:
    //retorna el id para hallar el virus creado
    static int createPatata(float x, float y, float r = 0.0f, float sx = 1.0f, float sy = 1.0f, float damage = 1.0f, float velocity = 0.4f)
    {
        int cTransformNextIndex = BufferManager::cTransformBuffer.size();
        BufferManager::cTransformBuffer.push_back(Transform {x, y, r, sx, sy});

        int cColliderNextIndex = BufferManager::cColliderBuffer.size();
        BufferManager::cColliderBuffer.push_back(Collider {30, 30});

        int cSpriteNextIndex = BufferManager::cSpriteBuffer.size();
        BufferManager::cSpriteBuffer.push_back(Sprite {sf::RectangleShape {}, sf::Texture {}});

        sf::Texture& txt = BufferManager::cSpriteBuffer[cSpriteNextIndex].txt;
        txt.loadFromFile("papa.png");

        sf::RectangleShape& rs = BufferManager::cSpriteBuffer[cSpriteNextIndex].rs;
        rs.setSize(sf::Vector2f {30.0f, 30.0f});
        rs.setPosition(sf::Vector2f {x, y});
        rs.setTexture(&txt, true);

        int cStatsNextIndex = BufferManager::cStatsBuffer.size();
        BufferManager::cStatsBuffer.push_back(Stats {damage, velocity});

        Virus p;
        p.transform = &BufferManager::cTransformBuffer[cTransformNextIndex]; // para hallar los componentes de la patata en la ultima posicion del buffer :v 
        p.collider  = &BufferManager::cColliderBuffer[cColliderNextIndex];
        p.sprite    = &BufferManager::cSpriteBuffer[cSpriteNextIndex];
        p.stats     = &BufferManager::cStatsBuffer[cStatsNextIndex];
        BufferManager::eVirusBuffer.push_back(p);

        return BufferManager::eVirusBuffer.size() - 1;
    }

private:
    VirusCreator() {}
};
```
### Estilo Trinity:
```C++
sf::RenderWindow window(sf::VideoMode(wx, wy), "Virus masheed");
sf::RectangleShape fondo(sf::Vector2f(wx, wy));

int miPatata1 = VirusCreator::createPatata(wx / 2 - tamPat / 2, wy / 2 - tamPat / 2);

while(window.isOpen())
{
    window.clear();
    VirusController::move(miPatata1);
    RenderingSystem::draw(window);

    sf::Event event;
    while (window.pollEvent(event))
    {
        if (event.type == sf::Event::Closed)
            window.close();
    }
    window.display();
}

return 0;
```
### Estilo Monolith:
```C++
 window.setEventHandler(eventHandler);

    glm::vec3 position = {0.0f, 0.5f, 8.0f};

    gil::Shader shader("3default");
    gil::setupDefaultLights(shader, position);

    gil::Model head("models/head.obj", nullptr, true, false);

    glm::vec3 headPos {0.0f, 0.0f, 0.0f};
    float moveWeight {2.0f};
    float yRotationAngle {-gil::constants::PI};
    float yRotationWeight {1.0f};

    gil::Timer timer {};
    glEnable(GL_DEPTH_TEST);
    while(window.isActive())
    {
        window.pollEvents();
        if(exit.isTriggered())
        {
            window.close();
            continue;
        }

        glClearColor(0.68f, 0.81f, 0.93f, 1.0f);
        glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);

        shader.use();

        float deltaTime = timer.getDeltaTime();
        headPos.x += xAxis.getMagnitude() * moveWeight * deltaTime;
        headPos.z += zAxis.getMagnitude() * moveWeight * deltaTime;
        yRotationAngle += yRot.getMagnitude() * yRotationWeight * deltaTime;

        glm::mat4 view = glm::lookAt(position, glm::vec3{0.0f, position.y, 0.0f}, glm::vec3{0.0f, 1.0f, 0.0f});
        glm::mat4 projection = glm::perspective(45.0f, window.getAspectRatio(), 0.1f, 256.0f);
        shader.setMat4("view", view);
        shader.setMat4("projection", projection);

        glm::mat4   model = glm::translate(glm::mat4(1.0f), headPos);
                    model = glm::rotate(model, yRotationAngle, glm::vec3{0.0f, 1.0f, 0.0f});
        shader.setMat4("model", model);
        head.draw(shader);

        window.swapBuffers();
    }
}
```
