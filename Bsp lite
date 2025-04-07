// ==UserScript==
// @name         Torn PDA Win Estimator — YATA Edition
// @namespace    http://tampermonkey.net/
// @version      2.0
// @description  Estimates faction member battle stats and win chance in Torn PDA based on YATA-style stat model
// @author       Code
// @match        https://www.torn.com/factions.php*
// @grant        GM_getValue
// @grant        GM_setValue
// ==/UserScript==

(function () {
    'use strict';

    const rankMultipliers = {
        "Absolute beginner": 0.4,
        "Beginner": 0.7,
        "Inexperienced": 1.0,
        "Intermediate": 1.4,
        "Competent": 1.8,
        "Distinguished": 2.0,
        "Professional": 2.4,
        "Veteran": 3.0,
        "Elite": 4.0,
        "Champion": 5.0,
        "Heroic": 6.0
    };

    const piratePrompt = () => {
        let wrapper = document.createElement('div');
        wrapper.style = `
            position: fixed; top: 0; left: 0; width: 100vw; height: 100vh;
            background-color: rgba(0, 0, 0, 0.85); z-index: 9999;
            display: flex; align-items: center; justify-content: center;
        `;

        wrapper.innerHTML = `
            <div style="background: #222; padding: 20px; border-radius: 10px; width: 90%; max-width: 350px; color: #fff; font-family: 'Verdana'; text-align: center;">
                <h3>Arrr! Enter ye Torn API Key:</h3>
                <input id="apiInput" placeholder="Your API Key" style="width: 100%; padding: 10px; border-radius: 5px; margin-top: 10px;">
                <button id="apiSave" style="margin-top: 15px; background: goldenrod; padding: 10px 20px; border: none; border-radius: 5px; font-weight: bold;">Set Sail</button>
            </div>
        `;
        document.body.appendChild(wrapper);
        document.getElementById('apiSave').onclick = () => {
            const key = document.getElementById('apiInput').value.trim();
            if (key.length >= 16) {
                GM_setValue('torn_api_key', key);
                location.reload();
            }
        };
    };

    const getOwnStats = async (apiKey) => {
        try {
            const res = await fetch(`https://api.torn.com/user/?selections=battlestats&key=${apiKey}`);
            const data = await res.json();
            return data && !data.error ? (data.strength + data.defense + data.dexterity + data.speed) : null;
        } catch (err) {
            console.error("Failed to fetch own stats:", err);
            return null;
        }
    };

    const getEnemyInfo = async (id, apiKey) => {
        try {
            const res = await fetch(`https://api.torn.com/user/${id}?selections=profile&key=${apiKey}`);
            const data = await res.json();
            if (data && !data.error) {
                return {
                    level: data.level,
                    rank: data.rank
                };
            }
            return null;
        } catch (err) {
            console.error(`Failed to fetch enemy info for ID ${id}:`, err);
            return null;
        }
    };

    const estimateStats = (level, rank) => {
        const base = 750;
        const multiplier = rankMultipliers[rank] || 1.0;
        return Math.floor(base * Math.pow(level + 5, 2) * multiplier);
    };

    const injectEstimate = async () => {
        const apiKey = GM_getValue('torn_api_key');
        if (!apiKey) return piratePrompt();

        const ownStats = await getOwnStats(apiKey);
        if (!ownStats) return piratePrompt();

        const members = [...document.querySelectorAll('a[href*="profiles.php?XID="]')];
        const already = new Set();

        for (let member of members) {
            const match = member.href.match(/XID=(\d+)/);
            if (!match) continue;

            const id = match[1];
            if (already.has(id)) continue;
            already.add(id);

            const container = member.closest('li') || member.parentElement;
            if (!container) continue;

            const enemy = await getEnemyInfo(id, apiKey);
            if (!enemy) continue;

            const enemyTotal = estimateStats(enemy.level, enemy.rank);
            const winChance = Math.floor((ownStats / (ownStats + enemyTotal)) * 100);
            const color = winChance >= 70 ? 'limegreen' : winChance >= 50 ? 'orange' : 'crimson';

            const box = document.createElement('div');
            box.innerHTML = `
                <div style="font-size: 11px; background: ${color}; color: black; padding: 2px 5px; margin-top: 2px; border-radius: 4px;">
                    ${winChance}% chance<br>~${enemyTotal.toLocaleString()} BS
                </div>
            `;
            box.style.marginLeft = '6px';
            container.appendChild(box);

            await new Promise(res => setTimeout(res, 1000)); // cooldown for API
        }
    };

    const waitAndRun = () => {
        const check = setInterval(() => {
            const members = document.querySelectorAll('a[href*="profiles.php?XID="]');
            if (members.length > 0) {
                clearInterval(check);
                injectEstimate();
            }
        }, 1000);
    };

    waitAndRun();
})();
