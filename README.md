#include <Geode/Geode.hpp>
#include <Geode/modify/PlayLayer.hpp>
#include <Geode/modify/PauseLayer.hpp>

using namespace geode::prelude;

struct ShowcaseRecord {
    std::string level;
    int difficulty;
    int attempt;
    std::string date;
    bool completed;
};

class $modify(ShowcasePlayLayer, PlayLayer) {
    bool init(GJGameLevel* level, bool useReplay, bool dontCreateObjects) {
        if (!PlayLayer::init(level, useReplay, dontCreateObjects))
            return false;

        return true;
    }

    void destroyPlayer(PlayerObject* player, GameObject* object) {
        if (m_level) {
            auto save = Mod::get()->getSaveContainer();

            auto attempts = save->getInt("lostAttempts", 0);
            save->setInt("lostAttempts", attempts + 1);
            save->saveData();
        }

        PlayLayer::destroyPlayer(player, object);
    }

    void levelComplete() {
        if (m_level) {
            auto save = Mod::get()->getSaveContainer();

            auto showcases = save->getInt("showcases", 0);
            save->setInt("showcases", showcases + 1);
            save->saveData();
        }

        PlayLayer::levelComplete();
    }
};

class $modify(ShowcasePauseLayer, PauseLayer) {
    bool init(bool checkpoint, bool recording) {
        if (!PauseLayer::init(checkpoint, recording))
            return false;

        auto menu = this->getChildByID("right-menu");

        if (menu) {
            auto trashButton = CCMenuItemSpriteExtra::create(
                ButtonSprite::create(
                    "🗑",
                    30,
                    true,
                    "bigFont.fnt",
                    "GJ_button_01.png",
                    30.0f,
                    1.0f
                ),
                this,
                menu_selector(ShowcasePauseLayer::onLostAttempts)
            );

            trashButton->setID("lost-attempts-button");
            menu->addChild(trashButton);

            auto cameraButton = CCMenuItemSpriteExtra::create(
                ButtonSprite::create(
                    "📹",
                    30,
                    true,
                    "bigFont.fnt",
                    "GJ_button_01.png",
                    30.0f,
                    1.0f
                ),
                this,
                menu_selector(ShowcasePauseLayer::onShowcases)
            );

            cameraButton->setID("showcases-button");
            menu->addChild(cameraButton);

            menu->updateLayout();
        }

        return true;
    }

    void onLostAttempts(CCObject*) {
        auto save = Mod::get()->getSaveContainer();

        int attempts = save->getInt("lostAttempts", 0);

        FLAlertLayer::create(
            "INTENTOS PERDIDOS",
            fmt::format(
                "Intentos perdidos: {}",
                attempts
            ),
            "OK"
        )->show();
    }

    void onShowcases(CCObject*) {
        auto save = Mod::get()->getSaveContainer();

        int showcases = save->getInt("showcases", 0);

        FLAlertLayer::create(
            "SHOWCASES",
            fmt::format(
                "Showcases completados: {}",
                showcases
            ),
            "OK"
        )->show();
    }
};{
    "geode": "$GEODE_VERSION",
    "gd": {
        "android": "2.2081"
    },
    "id": "eliasfd.showcase-recording",
    "name": "ShowCase Recording",
    "version": "1.0.0",
    "developer": "Eliasfd",
    "description": "Guarda intentos perdidos y showcases completados desde la pausa.",
    "tags": [
        "utility",
        "interface",
        "offline"
    ]
}
